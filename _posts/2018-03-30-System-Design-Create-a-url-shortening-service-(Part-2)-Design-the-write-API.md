---
layout: post
title:  "System Design: Create a url shortening service (Part 2): Design the write API"
categories: [Node.js, JavaScript, API]
author: Abhinav Dhasmana
---

This is part of a blog series where we design, develop, optimize, deploy and test a URL shortener service from scratch

*   Part 1: [Overview]({% post_url 2018-03-29-System-Design-Create-a-url-shortening-service-(Part-1)-Overview %})
*   **Part 2: Design the write API**
*   Part 3: [Read API, Load testing and Performance improvement]({% post_url 2018-03-31-System-Design-Create-a-url-shortening-service-(Part-3) %})
*   Part 4: [Deploy to AWS]({% post_url 2018-04-01-System-Design-Create-a-url-shortening-service-(Part-4) %})
*   Part 5: [Performance testing on AWS]({% post_url 2018-04-02-System-Design-Create-a-url-shortening-service-(Part-5) %})

In this article, we’ll discuss:

*   [API signature](#372c)
*   [How short should the short url be?](#ed91)
*   [How to generate short code](#4705)
*   [How to save the short code](#5172)

* * *

API signature is simple

`shorten(longUrl) // Returns a unique short url`

* * *

### How short should the short url be?

We can use the following characters in our short code:

*   A-Z(26)
*   a-z(26)
*   0–9(10)
*   _, -(2)

A total of 64\. With 6 characters, something like [http://ad.com/abcdef,](http://ad.com/abcdef,) we will be able to store 64⁶ unique urls. This is more than 68 billion (68,719,476,736). At 100 requests per second, it would take 21 years for this limit to exhaust.

* * *

### How to generate short code?

There are several ways to do it. One way would be to generate a random 6 character string. The problem with random is that if different people want to use the service with same longUrl, the system will generate different code and store it multiple times. This would be wastage of space. To solve this, let’s use hashing, specifically [MD5](https://en.wikipedia.org/wiki/MD5) hash. This would generate the same hash for the same input. We’ll use [base64](https://en.wikipedia.org/wiki/Base64#Base64_table) encoding on the hash to generate the string and take the first 6 characters. However, there is one catch. [Base64](https://en.wikipedia.org/wiki/Base64#Base64_table) is not url safe as it contains `/`and `+`. So we’ll replace these characters from Base64 encoding.

Here is the code snippet in Node.js

{% gist faa075e77a705a8a604eef7615399cc2 %}

* * *

### How to save the short code

As discussed earlier, we’ll use postgres. Other DBs are fine as well.

Database needs two fields: `code` to save the 6 character hash and the corresponding `originalurl`.

![](/images/blog/system-design-2/1.png){: .center-image }

The following migration will generate the above schema

{% gist ca3cb8a98efd4e54a04d980324963427 %}

`unique` constraint on the `code` creates index. Database part is done.

There can be several node servers running. We need to solve for concurrent writes. There can be a [race condition](https://stackoverflow.com/questions/4069718/postgres-insert-if-does-not-exist-already/13342031#13342031). We’ll use `[findOrCreate](http://docs.sequelizejs.com/manual/tutorial/models-usage.html#-findorcreate-search-for-a-specific-element-or-create-it-if-not-available)` to insert into postgres. If you are using noSQL database, find out how one can make an atomic transaction.

![](/images/blog/system-design-2/2.png){: .center-image }

Model code should look like this

{% gist e9460b1bbb366f8d0a2417e95dc003b3 %}

Since we are only using first 6 characters of the md5 hash, it is possible that we can get the same short code for two different long URLs. In order to resolve this, we’ll take next 6 characters from our base64 code and use them.

Here is the code which, when in conflict, would fetch the next 6 characters from the hash and use that for the short url

{% gist 2d8e7351e933a65f5168b729d8819746 %}

The only thing remaining now is to call this function from the routes.

{% gist 03e3940fc9ed4300764edd802dd13956 %}

If you are new to Hapi, here is how you can write your main server

{% gist d1381ca3d1c859a7edd93008214987e1 %}

The complete code can be found on [Github](https://github.com/abhinavdhasmana/tinyUrl).

* * *