---
layout: post
title:  "System Design: Create a url shortening service (Part 5): Performance testing on AWS"
tags: [Node.js, AWS, JMeter, Redis]
author: Abhinav Dhasmana
---

This is part of a blog series where we design, develop, optimize, deploy and test a URL shortener service from scratch.

*   Part 1: [Overview]({% post_url 2018-03-29-system-design-create-a-url-shortening-service-part-1-overview %})
*   Part 2: [Design the write API]({% post_url 2018-03-30-system-design-create-a-url-shortening-service-part-2-design-the-write-api %})
*   Part 3: [Read API, Load testing and Performance improvement]({% post_url 2018-03-31-system-design-create-a-url-shortening-service-part-3-read-api-load-testing-and-performance %})
*   Part 4: [Deploy to AWS]({% post_url 2018-04-01-system-design-create-a-url-shortening-service-part-4-deploy-to-aws %})
*   **Part 5: Performance testing on AWS**

* * *

In this article, we’ll talk about:

*   [Install and run JMeter on AWS](#5e6e)
*   [Few things that are not handled and some hints on how to do it](#9666).

* * *

### Install and run JMeter

To avoid latency in the result, Let’s spin up another EC2 instance (T2 Medium) in the same region.

Install JMeter

{% gist 1aa3fbb29a02a086f0ecddc5dcc54fd4 %}

Change the heap size of JMeter. Default is 512Mb. I have put it as 80% of RAM. `vi apache-jmeter-4.0/bin/jmeter` . Change heap size : `“${HEAP:=”-Xms4g -Xmx6g -XX:MaxMetaspaceSize=256m”}”`

Also `git clone` the repo as it has JMeter test plan. Important settings are:

*   250 Users with 250 sec ramp up time
*   Constant Throughput Timer at 60K requests/minute
*   I have 4 EC2 instances running before starting the JMeter test. Easiest way to achieve this is to change the `Desried` and `Min` setting in auto scaling group.

Start JMeter tests`./apache-jmeter-4.0/bin/./jmeter -n -t tinyUrl/tinyUrl.jmx`

After certain time, EC2 instances would start coming up. For me, AWS peaked at 8 instances

![](/images/blog/system-design-5/1.png){: .center-image }

I ran the test for ~19 minutes with more than a total of 10 million request.

![](/images/blog/system-design-5/3.png){: .center-image }

One of the target we set for ourself at the start of this series was the ability to serve 10K request/second. From the above log we can see we are able to serve that 👍

`[scp](https://www.ssh.com/ssh/scp/)` the log file `scp -i “<your pem file>” ubuntu@<ip where JMeter is running>:/home/ubuntu/perf.jt` in your box. Load it into the JMeter GUI.

![](/images/blog/system-design-5/4.png){: .center-image }

We fulfilled our target of responding to 90% of all the request in less than 10 ms 👍

Stop the JMeter and EC2 instances would start shutting down till they reach the `Min` setting in the auto scaling group.

We can also look at the `Activity History` inside `Auto Scaling Groups`

![](/images/blog/system-design-5/5.png){: .center-image }

* * *

### Other Observations

**100% cache hit rate**: All of the data is coming from Redis which is really fast and we have 100% cache hit ratio. In real life, it would not be the case. This would affect our performance. If the postgres data size is greater than what redis can store, it would impact the performance of the application as well.

**Read heavy application:** We have only performed performance test on read as this type of application is supposed to be read heavy.

**Scaling write heavy applications is harder:** Write heavy applications are harder to scale. A typical workflow would look like

*   Do vertical scaling
*   Separate read server from write servers
*   Configure the database in master-master config
*   If possible, write async way.

**Redundancy:** We have many single point of failures.

*   Single postgres server
*   Single Redis
*   Complete application is hosted in the same region of AWS

**Latency:** Users using this application from other regions of the world will face higher response time because of the latency.

**Automation:** Currently a lot of steps are manual in this setup. We can use [Chef](https://www.chef.io/chef/)/[Puppet](https://puppet.com/) etc to automate this whole process

* * *
