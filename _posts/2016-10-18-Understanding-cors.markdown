---
layout: post
title:  "Understanding same-origin policy and enable CORS for local development with react"
categories: [JavaScript, CORS, React.js, proxy server, Node.js]
summary: Understand CORS and use haproxy or webproxy to solve it
author: Abhinav Dhasmana
---

Lets consider the following scenario where a malicious website is trying to cheat you by faking transactions to your bank website

![CORS](/images/blog/understanding-cors/cors1.png){: .center-image }

In step 4: A malicious website is reading cookies from some other website. This is bad as once malicious website has this data, it cannot differentiate between the good user and a fake user.

To stop this from happening, browsers implement same-origin policy. What this basically means is that to make a second request, first being the home page, the second request must be from the same origin as the destination.


![CORS rules](/images/blog/understanding-cors/cors2.png){: .center-image }

In local development, same-origin policy can be an issue. Consider this scenario:

* Our webpack is running on port 3000
* Our node server is running on port 4000
When we go to http://localhost:3000 and this page makes a request for port 4000, it would go into preflight (more on preflight [here] and [wiki]).


To avoid this, there are several solutions we can use:

* [Chrome plugin to allow CORS]
* Write custom code in your app for preflight to allow CORS for certain domains
* Use proxies

---

My personal preference is proxies as it requires little code. If written right, this change has little chance to affect your production code.

Option 1: HAProxy

On the mac, you can install haproxy via brew

```brew install haproxy```

This is how my haproxy file looks like at the root of our react project

{% gist f18ca671e53611f6798a1f8497b7e9de %}

Once this is done, we need to start 3 servers:

* our node api running at port 4000
* our webpack at port 3000
* Start haproxy server with haproxy -f proxy.cfg from root of the react folder

Now if we visit http://localhost:9000 haproxy will handle CORS for us.

Option 2: Built in proxies

Many boiler plates come with proxies as part of the code. We are using [React-Redux-Starter-Kit] and this uses [koa-proxy]. All we have to do is enable it. Following are the contents of the file at “config/_development.js”

{% gist be0a62763f6475b7a3edc365704b85d4 %}

Now all we have to do is

* start our node api running at port 4000
* start our webpack at port 3000


Now if we visit http://localhost:3000, things would work with no issues.


[here]: https://www.html5rocks.com/en/tutorials/cors/
[wiki]: https://en.wikipedia.org/wiki/
[Chrome plugin to allow CORS]: https://chrome.google.com/webstore/detail/allow-control-allow-origi/nlfbmbojpeacfghkpbjhddihlkkiljbi?hl=en
[koa-proxy]: https://www.npmjs.com/package/koa-proxy
[React-Redux-Starter-Kit]:https://github.com/davezuko/react-redux-starter-kit