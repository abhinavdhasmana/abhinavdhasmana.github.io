---
layout: post
title:  "System Design: Create a url shortening service (Part 3): Read API, Load testing and performance optimization"
categories: [Node.js, sequelize, postgres, Jmeter]
author: Abhinav Dhasmana
---
This is part of a blog series where we design, develop, optimize, deploy and test a URL shortener service from scratch

*   Part 1: [Overview]({% post_url 2018-03-29-System-Design-Create-a-url-shortening-service-(Part-1)-Overview %})
*   Part 2: [Design the write API]({% post_url 2018-03-30-System-Design-Create-a-url-shortening-service-(Part-2)-Design-the-write-API %})
*   **Part 3: Read API, Load testing and Performance improvement**
*   Part 4: [Deploy to AWS]({% post_url 2018-04-01-System-Design-Create-a-url-shortening-service-(Part-4) %})
*   Part 5: [Performance testing on AWS]({% post_url 2018-04-02-System-Design-Create-a-url-shortening-service-(Part-5) %})

In this article, we’ll discuss:

*   [Read API](#4224)
*   [Seed the data to validate code](#7e67)
*   [Load testing using JMeter](#be28)
*   [Optimization](#3da6)

* * *

### Read API

Read API is pretty simple. Given a short code, returns the original url.

`longUrl(minifiedURL) // Returns the long url that was encoded`

Updated model to read from the database

{% gist d814802104bc379a4ab1dd3d868512ec %}

Add a new route to get this data

{% gist 7f3bf4d487dd99f322c18a25dcf0330d %}

* * *

### Seed the data

We’ll enter a million entries into the database to start with. We’ll use [Sequelize seeders](https://github.com/sequelize/cli#documentation)

{% gist 172dd6047472143c443b755b4976ec9d %}

We can validate the code by making this `GET` request `[http://localhost:3000/longUrl?code=Pl1pM](http://localhost:3000/longUrl?code=Pl1pMh)h` and should get `{“originalUrl”:”http://www.thisissomereallylongurl1"}` as response.

* * *

### Load testing using JMeter

If you are not familiar with JMeter, we will create test plan and configure it.

Step 1: Add users

![](/images/blog/system-design-3/1.png){: .center-image }

Let’s call this read url. Add 100 users. I have set this up as below

![](/images/blog/system-design-3/2.png){: .center-image }

It’s important to note that I have given 50 seconds as a ramp up time. This means that number of threads starting at each second is 100(users)/50 = 2\. Since JMeter and node server is running on the same hardware, this would affect results if this number is low.

Let’s control the maximum number of requests coming in to the server. Add `Constant Throughput Timer` for 100 requests/second

![](/images/blog/system-design-3/3.png){: .center-image }
< br/ >
![](/images/blog/system-design-3/3_7.png){: .center-image }

Now, before we make request, we want some random short url for which we want to fetch the long url. We can create a CSV file and let JMeter read it recursively.

![](/images/blog/system-design-3/3_5.png){: .center-image }

Fill in the details as below

![](/images/blog/system-design-3/5.png){: .center-image }

and `code2.csv` is pretty simple

![](/images/blog/system-design-3/6.png){: .center-image }

Next: Add HTTP request and read from the csv file.

![](/images/blog/system-design-3/7.png){: .center-image }
<br />
![](/images/blog/system-design-3/8.png){: .center-image }
<br />

Add a listener to display the result.
<br />

![](/images/blog/system-design-3/9.png){: .center-image }

That’s it.

Below are the results I got on my mac

![](/images/blog/system-design-3/10.png){: .center-image }

I have other apps running along with this which could affect the performance. This still gives us a good enough estimate of how well the app will perform in the wild with 100 requests per second.

We’ll focus on `90% line` and `95% line` in the above table and optimize it.

* * *

### Optimization

**Optimization 1: W**e can play with database connections pool size. We can edit our `config/config.json` file and increase our connection pool

<pre name="fef4" id="fef4" class="graf graf--pre graf-after--p">"development": {

"username": "abhinavdhasmana",
"password": null,
"database": "tinyurl_development",
"host": "127.0.0.1",
"dialect": "postgres",
"pool": {
  "max": 20,
  "min": 5,
  "idle": 10000
}
</pre>
Below are the results:

![](/images/blog/system-design-3/11.png){: .center-image }

Metrics improved by roughly 20%. At this point of time, I am not sure what is the magic number on `max` or `min` on the pool connection. If you know more, please leave a comment.

**Optimization 2**: Node.js is single threaded. One node server cannot use all the CPU cores. This is hardware wastage. Welcome [PM2](http://pm2.keymetrics.io/). PM2 can run one node process per CPU core. Its simple to setup and start

<pre name="07b4" id="07b4" class="graf graf--pre graf-after--p">npm install pm2
node_modules/.bin/pm2 start -i max src/server.js</pre>

![](/images/blog/system-design-3/12.png){: .center-image }

Wow! Now I was able to get 50% and 70% improvement on `90% line` and `95% line` respectively.

**Optimization 3:** The slowest part in the application is the hard disk. Lets minimize its use. We will put this data in [R](https://redis.io/)edis. Redis is an in-memory data structure store. Code is modified below to use Redis

{% gist 11bf085de74ad249230c862b9caaa5d6 %}

and run our load test again

![](/images/blog/system-design-3/13.png){: .center-image }

This is fabulous now. I was able to get over 70% and 85% improvement on `90% line` and`95% line` respectively

That’s it for the performance optimization. App is now ready to be deployed and be tested in the wild.

The complete code can be found on [Github](https://github.com/abhinavdhasmana/tinyUrl).

* * *
