---
layout: post
title:  "How to seed test database in node"
categories: Node.js
author: Abhinav Dhasmana
---
Writing test cases is critical to any project. Any real test cases would require the test case to talk to the database. Seeding the database is a standard practice which allows the test cases to run and validate on top of the existing data.

Here is my scenario:

* I have a seed.sql file that has raw sql statements that fills in the data.
* I am using [db-migrate][db-migrate] to manage database migrations
* I need to reset my database and run all migrations once every time my test cases run.


**Step 1: Reset the database and run migrations before we run tests.**
In my `package.json`, I added a new task


```
"testdb": "db-migrate reset --env test && db-migrate up --env test"
```
Now all we have to do is run this before every test. I already have a test script which looks like this

```
"test": "./node_modules/.bin/gulp"
```

which can be modified to

````
"test": "npm run testdb && ./node_modules/.bin/gulp",
````

So my package.json script looks like this

```
scripts: {
    "test": "npm run testdb && ./node_modules/.bin/gulp",
    "testdb": "db-migrate reset --env test && db-migrate up --env test"
  },
```



**Step 2: Prepare a script to run before you run test cases.**
In `tests.js` in the `gulp` folder, change your gulp tasks to run the `bootstrapTest.js`.

Original code:

```
gulp.task('test', ['lintSources', 'lintTests'], function () {
  return gulp.src(['test/**/*.js'])
    .pipe(mocha({
      reporter: 'mocha-junit-reporter',
      reporterOptions: {
        mochaFile: CIRCLE_TEST_REPORTS + '/junit/results.xml'
      }
    }))
    .pipe(istanbul.writeReports())
    .once('error', (e) => {
      console.error(e);
      process.exit(1);
    })
    .once('end', () => {
      process.exit();
    });
});
```

New code

```
gulp.task('test', ['lintSources', 'lintTests'], function () {
  return gulp.src(['test/bootstrapTest.js', 'test/**/*.js'])
    .pipe(mocha({
      reporter: 'mocha-junit-reporter',
      reporterOptions: {
        mochaFile: CIRCLE_TEST_REPORTS + '/junit/results.xml'
      }
    }))
    .pipe(istanbul.writeReports())
    .once('error', (e) => {
      console.error(e);
      process.exit(1);
    })
    .once('end', () => {
      process.exit();
    });
});
```

**Step 3: Setup `bootstrapTest.js`**

```
const exec = require('child_process').execSync;
exec('./seed.sh', {env: {database__host: 'localhost', database__user: 'user', database__password: 'password', database__port: 3306, database__name: 'testdb'}});
```

Above code runs the `seed.sh` file synchronously. Here is my sample `seed.sh` file

```
#!/bin/bash
mysql --host $database__host --user $database__user --port $database__port --password=$database__password $database__name < seed.sql
echo '## Database seeded'
```

and here is my sample seed.sql file

```
INSERT INTO users
  (email, id)
VALUES
  ('email1@email.com', 1);

INSERT INTO users
  (email, id)
VALUES
  ('email2@email.com', 2);

INSERT INTO users
  (email, id)
VALUES
  ('email3@email.com', 3);

INSERT INTO users
  (email, id)
VALUES
  ('email4@email.com', 4);

```











{% if true %}
  <div id="disqus_thread"></div>
  <script>
    var disqus_config = function () {

    this.page.url = "http://www.abhinavdhasmana.in/node/2016/03/28/2016-03-28-seed-test-database-with-nodejs.html"; // Replace PAGE_URL with your page's canonical URL variable
    this.page.identifier = "node/2016/03/28/2016-03-28-seed-test-database-with-nodejs.html";
    };

    (function() { // DON'T EDIT BELOW THIS LINE
      var d = document, s = d.createElement('script');
      s.src = '//abhinavdhasmana.disqus.com/embed.js';
      s.setAttribute('data-timestamp', +new Date());
      (d.head || d.body).appendChild(s);
      })();
  </script>
  <noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
{% endif %}

[db-migrate]: https://www.npmjs.com/package/db-migrate