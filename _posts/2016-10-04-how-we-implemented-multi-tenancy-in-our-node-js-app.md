---
layout: post
title:  "How we implemented multi-tenancy in our Node.js App"
tags: [Node.js, JavaScript, Sequelize.js, Multitenancy]
summary: How to convert a single tenant application into multi-tenant with Sequelize.js
author: Abhinav Dhasmana
---

We started our application as a single tenant application. This is how the basic digram would look like.

![Single tenant application](/images/blog/multi-tenancy/multi-tenancy-1.png){: .center-image }

Now we had to make this application multi-tenant. We considered the following options

Option 1: Physical separation of data

![Multi-Tenancy with DB Connector Service](/images/blog/multi-tenancy/multi-tenancy-2.png){: .center-image }

In this scenario, we would write a new micro service (or with the current app) which would parse the incoming request and based on the tenant information passed along, will decide which database to connect to.

Option 2: Logical separation of data

![Multi-tenancy with logical separation of data](/images/blog/multi-tenancy/multi-tenancy-3.png){: .center-image }

In this scenario, there is just one single database, and all the responsibilities lies within our app for all the CRUD operations.

---

We decided to go with Option 2 because of the following reasons:

* Easy client on-boarding: New client on-boarding is easier as we do not need to provision a new database for every new client.
* No need to change in the “backend” code for every new client. Option 1 would require change in the DB Connector service.
* Creating client specific react app allowed us to do UI branding for the client.


The obvious problem that we see with Option 2 is that its far easier to make mistakes and introduce bugs into the system as every request have to query the right set of rows and not mix up. This may seem like a lot of work but its not. Here’s how:

---

Step 1: Write a migration to create a field called `tenant_id` to the tables which you want to be multi-tenant. We are using [db-migrate] to manage our migrations. Here is a sample migration
```javascript

const async = require(‘async’);
const columnName = ‘tenant_id’;
const columnSpec = {
 type: ‘int’,
};
exports.up = function (db, callback) {
 async.series([
 db.addColumn.bind(db, ‘table1’, columnName, columnSpec),
 db.addColumn.bind(db, ‘table2’, columnName, columnSpec),
 db.addColumn.bind(db, ‘table3’, columnName, columnSpec)
], callback);
exports.down = function (db, callback) {
 async.series([
db.removeColumn.bind(db, ‘table1’, columnName),
 db.removeColumn.bind(db, ‘table2’, columnName),
 db.removeColumn.bind(db, ‘table3’, columnName)
], callback);

```
Don’t forget to add indexes as they will always be queried upon.

Step 2: Next we want to make sure all our [Sequalize] models use tenant_id and make it transparent to all the developers working on the project.

This is where scope comes in specially defaultScope.

Lets say your old model looks like this

``` javascript
const a2 = sequelize.define(‘users’, {
 id: {type: Sequelize.INTEGER, primaryKey: true, autoIncrement: true, unsigned: true},
 username: Sequelize.STRING(50),
 email: Sequelize.STRING(50),
 });
 ```
And new one would look like this

``` javascript
const a2 = sequelize.define(‘users’, {
 id: {type: Sequelize.INTEGER, primaryKey: true, autoIncrement: true, unsigned: true},
 username: Sequelize.STRING(50),
 email: Sequelize.STRING(50),
 tenant_id: Sequelize.INTEGER,
 }, {
    defaultScope: {
      where: {
        tenant_id: tenantId, // tenantId is passed to the function which initializes the object and this is the only way to create an object within the app
      },
    },
  });
  ```
What we have done here is added a defaultScope to the model and passed a tenantID variable to object creation function. Now every time we create an object with a valid tenantId, we do not have to worry about the querying on the right tenant as this is in the default scope.

Step 3: Scopes apply to

``` javascript
.find, .findAll, .count, .update and .destroy.
```
We still have to worry about the new inserts. For inserts we use hooks. Adding 3 lines of code, does the trick

``` javascript
users.hook(‘beforeValidate’, function (model) {
 model.tenant_id = tenantId;
 });
 ```
So, this is how the full model/dao looks like

``` javascript
const initializeUsers = function (tenantId) {
 const users = sequelize.define(‘users’, {
 id: {type: Sequelize.INTEGER, primaryKey: true, autoIncrement: true, unsigned: true},
 username: Sequelize.STRING(50),
 email: Sequelize.STRING(50),
 tenant_id: Sequelize.INTEGER,
 }, {
 defaultScope: {
 where: {
 tenant_id: tenantId,
 },
 },
 });
users.hook(‘beforeValidate’, function (model) {
 model.tenant_id = tenantId;
 });
 return users;
};
module.exports = initializeUsers;
```

[db-migrate]: https://github.com/db-migrate/node-db-migrate
[Sequalize]: https://sequelize.readthedocs.io/en/v3/