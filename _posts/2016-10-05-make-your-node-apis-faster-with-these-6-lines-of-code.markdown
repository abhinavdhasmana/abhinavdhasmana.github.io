---
layout: post
title:  "Make your Node APIs faster with these 6 lines of code"
categories: [Node.js, JavaScript, Performance, Cache]
summary: Use in memory cache to boost your api performance
author: Abhinav Dhasmana
---


![Making things simple](/images/blog/in-memory-cache/in-memory-cache.png){: .center-image }

It is hard to make something simple.

I was set out to make our node app more performant. Performance is hard and complicated and time consuming and did I mention hard?

So, I opted for in memory caching. This is what my code looked like before caching

``` javascript
module.exports = function () {
 return {
   method: ‘GET’,
   path: ‘/myGetRequest/{myParam}’,
   handler: function (request, reply) {
     const requestParam = decodeURIComponent(request.params.myParam);
     const result = myActualCodeToGetData(requestParam);
     return reply(result);
   },
 };
};
```

With the use of memory-cache, we could implement caching with just few lines (in bold).

``` javascript
const cache = require('memory-cache');
module.exports = function () {
 return {
   method: ‘GET’,
   path: ‘/myGetRequest/{myParam}’,
   handler: function (request, reply) {
     const requestParam= decodeURIComponent(request.params.myParam),
       key = 'myGetRequest' + myParam.toLowerCase(),
       cachedResult = cache.get(key);
     if (cachedResult !== null) {
        return reply(cachedResult);
      }
     const result = myActualCodeToGetData(requestParam);
     cache.put(cacheKey, result);
     return reply(result);
   },
 };
};
```

Adding just caching, we were able to reduce our most complex request from 700ms to less than 50ms.

Of course, you would need a way to invalidate your cache, which is as simple as putting it in cache

``` javascript
cache.clear() // deletes all keys
cache.del(key) // deletes a particular
```

I know, the following questions would be coming to your mind:

* All my caching goes for a toss whenever my server restarts?
* What if my cache key is unique for most of my requests?
* Do I have enough RAM on my box?
* If I am sitting behind a load-balancer with many servers, what would be my cache-hit ratio?
* My App is more UPSERT than read


Sure, in the above scenarios and may be more, this might not make sense. There you can go with Redis/memcached.