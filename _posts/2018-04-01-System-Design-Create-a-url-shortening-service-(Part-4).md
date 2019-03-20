---
layout: post
title:  "System Design: Create a url shortening service (Part 4): Deploy to AWS"
categories: [Node.js, AWS, postgres, Redis]
author: Abhinav Dhasmana
---

This is part of a blog series where we design, develop, optimize, deploy and test a URL shortener service from scratch.

*   Part 1: [Overview]({% post_url 2018-03-29-System-Design-Create-a-url-shortening-service-(Part-1)-Overview %})
*   Part 2: [Design the write API]({% post_url 2018-03-30-System-Design-Create-a-url-shortening-service-(Part-2)-Design-the-write-API %})
*   Part 3: [Read API, Load testing and Performance improvement]({% post_url 2018-03-31-System-Design-Create-a-url-shortening-service-(Part-3) %})
*   **Part 4: Deploy to AWS**
*   Part 5: [Performance testing on AWS]({% post_url 2018-04-02-System-Design-Create-a-url-shortening-service-(Part-5) %})

* * *

In this article, we’ll talk about:

*   [Overall architecture of our app](#3dc7)
*   [Software installation and settings](#d968)
*   [Create load balancer on AWS](#d0e9)
*   [Create auto scaling groups on AWS and connect to the load balancer](#5dbd)

* * *

### Overall architecture

Our tiny service is now ready to be deployed. Our architecture would look like this

![](/images/blog/system-design-4/1.png){: .center-image }

We’ll go for [horizontal scaling instead of vertical scaling](https://stackoverflow.com/questions/11707879/difference-between-scaling-horizontally-and-vertically-for-databases). For this small app , we would not use read replicas or master-slave configuration.

We’ll use AWS for setting up our boxes with t2 medium instance type with Ubuntu OS for Node and t2 micro instances for postgres and redis.

* * *

### Installation steps

**Redis:** Start an EC2 instance with Ubuntu image. `ssh` into the box and install redis.

<pre name="06fd" id="06fd" class="graf graf--pre graf-after--p">sudo apt-get update
sudo apt-get install redis-server</pre>

Now redis server is running but does not accept incoming connections from other IPs. Lets edit the conf file `sudo vi /etc/redis/redis.conf` . Change `bind` to `bind 0.0.0.0` and restart the redis server `sudo /etc/init.d/redis-server restart` .

**Postgres:** Start an EC2 instance with Ubuntu image. `ssh` into the box and install postgres.

<pre name="00a7" id="00a7" class="graf graf--pre graf-after--p">1  sudo apt-get update
2  sudo apt-get install postgresql postgresql-contrib</pre>

Create a role in postgres which our app would be using to connect from our node server. Make sure to change `config/config.json` in the node code file so that the app can connect to the database.

Like redis, change the config to accept incoming traffic. Edit `sudo vi /etc/postgresql/9.5/main/postgresql.conf` and change listen_address to `listen_address = '*'` and restart.

_P.S: These are not the best way to do as we are opening our servers to connect from any IP. Ideally it should only be a subset of IPs on which our node app is running._

**Node server:** Start an EC2 instance with Ubuntu image. `ssh` into the box and follow the below steps

1.  `git clone [https://github.com/abhinavdhasmana/tinyUrl.git](https://github.com/abhinavdhasmana/tinyUrl.git)`
2.  Ensure that the changes in `config/config.json` matches the host entry, username and password entry for the postgres server . For example, my config looks like this

<pre name="d053" id="d053" class="graf graf--pre graf-after--li">{
  "development": {
    "username": "abhinavdhasmana",
    "password": "test",
    "database": "tinyurl_development",
    "host": "172.31.17.70",
    "dialect": "postgres",
    "pool": {
      "max": 20
    }
  },</pre>

I am using private IP of the box. Public IP would work as well. Prefer private over public as restarts changes the public IP but not the private.

3\. Change the redis config in `src/server.js` to point to the right IP. Again, this is the private IP

`const redisClient = redis.createClient({host: ‘172.31.17.28’});`

We can also create `redis.yml` and define different settings for connection based on different envs.

4\. Next, lets setup our database with the help of seqeulize and seed the data

`./node_modules/.bin/sequelize db:create`

`./node_modules/.bin/sequelize db:seed:all`

If the above command works, we are sure our database connection is working fine.

5\. Do `npm start` and our server is running.

* * *

### Create load balancer

![](/images/blog/system-design-4/2.png){: .center-image }

Select all the subnet. In the above image I have left `us-east-2c` or make sure you instance is launched in `2a` or `2b` only.

Next step, security group. All we need is port 80 from anywhere.

![](/images/blog/system-design-4/3.png){: .center-image }

Next step: Create a new target group. We will come to this when we do auto scaling. Node server is running on port 8080, hence its 8080\. Also we have a route `/ping` for our health check up which I have entered.

![](/images/blog/system-design-4/4.png){: .center-image }

Once we have a target group, any EC2 instance that launches in that group and once the instance passes the Healthy threshold, load balancer will start sending request to this instance.

* * *

### Create auto scaling groups

Let’s try to launch the EC2 instance and in step 3 of the configuration, click `launch into Auto Scaling Group`

![](/images/blog/system-design-4/5.png){: .center-image }

In the `Create Launch Configuration` screen, fill in the name and add the below script inside `User data` . This will get our server up and running as part of the boot process. Note that this script is run only once per instance and not on every restart. If you want to run it on every restart, delete `config_scripts_user` from below location

`sudo rm /var/lib/cloud/instances/*/sem/config_scripts_user`

{% gist e3849116edb5a539594ae3d0ce07bc2f %}

We need to install `nvm` and the app as the ubuntu user. So the above script creates a file and run this as a ubuntu user.

![](/images/blog/system-design-4/6.png){: .center-image }

Go through the flow and complete the configuration. Once done, it will redirect to the creation of auto scaling group.

Fill in the details as below. Make sure to fill `Target Groups` in the `Advanced Details`

![](/images/blog/system-design-4/7.png){: .center-image }

Next is we need to decide on _when_ should EC2 instances should be added and removed. This is pretty simple in AWS. In the image below, we want the system to add a new machine when average CPU utilization goes beyond 70%.

![](/images/blog/system-design-4/8.png){: .center-image }

Go through the flow to complete this and we have created an auto scaling group.

Give it some time and a new instance would be up and running which should be visible in the EC2 console.

That’s it. Now we need to know the DNS of the load balancer and we are good to go. In my case its `tinuUrlLoadBalancer-269014099.us-east-2.elb.amazonaws.com` , so I hit `[http://tinuurlloadbalancer-269014099.us-east-2.elb.amazonaws.com/ping](http://tinuurlloadbalancer-269014099.us-east-2.elb.amazonaws.com/ping)` and get `pong` as a response.

* * *
