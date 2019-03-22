---
layout: post
title:  "Everyday Software Design Patterns"
tags: [JavaScript,  Node.js, Design Patterns]
summary: Some basic examples of design patterns
author: Abhinav Dhasmana
---

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Frameworks and APIs change fast. Software design principles are evergreen. Learn principles that translate across language barriers.</p>&mdash; Eric Elliott (@_ericelliott) <a href="https://twitter.com/_ericelliott/status/728670743660273665?ref_src=twsrc%5Etfw">May 6, 2016</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">Best quote today: “software changes fast, in the past 5 minutes, 7 new JavaScript libraries were released!” -<a href="https://twitter.com/jeremybytes?ref_src=twsrc%5Etfw">@jeremybytes</a> LMAO!</p>&mdash; David Barkman (@cybler) <a href="https://twitter.com/cybler/status/784942699749122048?ref_src=twsrc%5Etfw">October 9, 2016</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>


<br/>

If you write software, you use design patterns knowingly or unknowingly. If we learn principles we use instead of just learning how the framework works, life can be little simpler.


Lets take a look at some of the very very basic examples that you would have used.

**Example 1: Database connection with Node.js and Hapi**

Lets say your server is connecting to the database. Its working fine and this is how your config/config.js

``` javascript
{
  "development": {
    "username": "myusername",
    "password": myuserpassword,
    "database": "mydatabase",
    "host": "127.0.0.1",
    "dialect": "postgres"
  }
}
```


Now lets change the database name without restarting the node server

``` javascript
{
  "development": {
    "username": "myusername",
    "password": myuserpassword,
    "database": "garbageDatabase", // changed the database name
    "host": "127.0.0.1",
    "dialect": "postgres"
  }
}
```

Now if we hit the server, what do we expect? Should our server responds with the error or with the response as if nothing has happened?

The correct answer is that the server would respond as if nothing has changed. Looks like the database connection is made when the server starts. If we think about it a little more it makes sense as making connection to db is [expensive]. Looks like [Singleton pattern] is used here.

Example 2: Memory leak in Node.js

Lets take a look at this code

{% gist 135067a3706f9dd59f6b9720ff0b00ba %}

All this code does is whenever a request is made to <host:3000>, its starts pushing an array into myBigFatArray. Lets try and simulate this server to a production server where you receive multiple request with many concurrent users. To do this we will use Apache Benchmark. We use the following commands:

Start server: node server.js

Start [apache benchmark]: ab -n 30000 -c 20 http://127.0.0.1:3000/

![memory leak](/images/blog/design-pattern/design-pattern.gif){: .center-image }

While running the benchmark you would eventually run out of memory and get this error

```
<--- Last few GCs --->
17065 ms: Mark-sweep 1042.2 (1457.9) -> 1042.1 (1457.9) MB, 162.8 / 0 ms [allocation failure] [scavenge might not succeed].
   17232 ms: Mark-sweep 1042.1 (1457.9) -> 1042.1 (1457.9) MB, 166.8 / 0 ms [allocation failure] [scavenge might not succeed].
   17407 ms: Mark-sweep 1042.1 (1457.9) -> 1042.1 (1457.9) MB, 175.0 / 0 ms [last resort gc].
   17587 ms: Mark-sweep 1042.1 (1457.9) -> 1042.1 (1457.9) MB, 180.5 / 0 ms [last resort gc].
<--- JS stacktrace --->
==== JS stack trace =========================================
Security context: 0x134324bb4629 <JS Object>
    2: /* anonymous */(aka /* anonymous */) [/Users/abhinavdhasmana/Documents/Personal/sourcecode/memoryLeak/server.js:~13] [pc=0x571bd8907f4] (this=0x134324b041b9 <undefined>,request=0x32a5bb2fbbd1 <JS Object>,reply=0x32a5bb2fdd81 <JS Function reply (SharedFunctionInfo 0x271b1d0a7f39)>)
    3: handler [/Users/abhinavdhasmana/Documents/Personal/sourcecode/memoryLeak/node_modules/hapi/lib/handle...
FATAL ERROR: CALL_AND_RETRY_LAST Allocation failed - process out of memory
[1]    46301 abort      node server.js
```

If we look at this problem, we will realize that the problem is our myBigFatArray. Looks like it is initialized once the lifetime of the server. If it would have been inside the handler , it would have been just fine as GC would have cleaned it up upon the end of each request. Another simple example of Singleton pattern.

There are some other patterns that are commonly used like [Observer] and [Module] but one pattern at a time


[expensive]: https://dba.stackexchange.com/questions/16969/how-costly-is-opening-and-closing-of-a-db-connection
[Singleton pattern]: https://en.wikipedia.org/wiki/Singleton_pattern
[apache benchmark]: https://httpd.apache.org/docs/2.4/programs/ab.html
[Observer]:https://en.wikipedia.org/wiki/Observer_pattern
[Module]:https://en.wikipedia.org/wiki/Module_pattern
