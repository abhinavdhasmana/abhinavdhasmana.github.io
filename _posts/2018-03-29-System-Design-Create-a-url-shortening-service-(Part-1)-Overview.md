---
layout: post
title:  "System Design: Create a url shortening service (Part 1): Overview"
categories: [AWS, software design, Software Development]
author: Abhinav Dhasmana
---

URL shortening service like [bit.ly](https://bitly.com/) or [goo.gl](https://goo.gl/) are hugely popular. They simplify link sharing, analytics and much more. We’ll create one feature of these apps. Create a short url from the long url and returns the original url of a short url.

Designing the “right” system is hard. “Right” keeps on changing. Requirements change. Traffic pattern changes. The only thing that is constant is change.

The “right” for this service is:

*   System should be able to store “enough” urls
*   System should handle 10K request per second
*   90 percent of all request should respond in less than 10ms for the read request
*   Save money when not receiving enough load

* * *

In the next few blog posts, We will create a system which:

1.  Given a long url, returns a short code for it.
2.  Given a short url, returns the long url if it exists or not found.
3.  Deploy this on AWS on a single EC2 instance.
4.  Auto scale the app based on the traffic.
5.  Load test the application.

Tech stack details:

*   Node.js using [Hapi server](https://hapijs.com/)
*   [Postgres](https://www.postgresql.org/) as database
*   [Sequelize](http://docs.sequelizejs.com/) as ORM
*   [Redis](https://redis.io/) for caching
*   [JMeter](https://jmeter.apache.org/) for load testing
*   [AWS](https://aws.amazon.com/) for hosting

The complete code can be found on [Github](https://github.com/abhinavdhasmana/tinyUrl).

This series is comprised of following parts:

*   **Part 1: Overview**
*   Part 2: [Design the write API]({% post_url 2018-03-30-System-Design-Create-a-url-shortening-service-(Part-2)-Design-the-write-API %})
*   Part 3: [Read API, Load testing and Performance improvement]({% post_url 2018-03-31-System-Design-Create-a-url-shortening-service-(Part-3) %})
*   Part 4: [Deploy to AWS]({% post_url 2018-04-01-System-Design-Create-a-url-shortening-service-(Part-4) %})
*   Part 5: [Performance testing on AWS]({% post_url 2018-04-02-System-Design-Create-a-url-shortening-service-(Part-5) %})



* * *