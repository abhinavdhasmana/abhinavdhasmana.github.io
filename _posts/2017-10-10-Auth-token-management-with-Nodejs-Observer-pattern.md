---
layout: post
title:  "Auth token management with Node.js Observer pattern"
categories: [Node.js, software design pattern, Authentication]
author: Abhinav Dhasmana
---


Consider the following situation where you are getting data from a third party server using the authentication token.

![](/images/blog/auth-token-management/1.png){: .center-image }

1.  You request the auth token
2.  Auth token received.
3.  Send request with auth token
4.  Received the desired response.

And once the token expires, we fetch a new one, save it and keeps on making this new request with this new auth token.

Here is the sample code with for request.

{% gist 6ed3c797d3525a925ddaeb8c1898b273 %}

Below is our mocked authentication system

{% gist 7fff05b7a090f1985e9f3ecc92f089ac %}

Now if we run both apps on respective port (8080 and 8081), and run this test script

{% gist 7e1f734289d3fbde78198fae3e8f8e7c %}

we would received the following logs at the services as expected

<pre name="30f0" id="30f0" class="graf graf--pre graf-after--p">// serviceHandler output
{“newToken”:”def”}
No need for new request
No need for new request
No need for new request
No need for new request
No need for new request
No need for new request
No need for new request
No need for new request</pre>

<pre name="7594" id="7594" class="graf graf--pre graf-after--pre">// Token generator output
I am handling token: abc</pre>

Lets go ahead and modify our test script to run the curl commands in parallel. This simulates multiple incoming request to your node server.

{% gist a9f82debb3fdf8248bf6337b11e926bf %}

And the output this time???

<pre name="09be" id="09be" class="graf graf--pre graf-after--p">// serviceHandler.js</pre>

<pre name="b8d3" id="b8d3" class="graf graf--pre graf-after--pre">{"newToken":"def"}
{"newToken":"def"}
{"newToken":"def"}
{"newToken":"def"}
{"newToken":"def"}
{"newToken":"def"}
{"newToken":"def"}
{"newToken":"def"}
{"newToken":"def"}</pre>

<pre name="041e" id="041e" class="graf graf--pre graf-after--pre">// tokenGenerator
I am handling token: abc
I am handling token: abc
I am handling token: abc
I am handling token: abc
I am handling token: abc
I am handling token: abc
I am handling token: abc
I am handling token: abc
I am handling token: abc</pre>

As you can see from the output, all request were made in parallel and all ended up calling the auth service.

_Wouldn’t it be nice if only one request was made to the auth service for getting the new token while other request wait for token generation to be completed. Once the authentication is completed, it should notify the other requests to use the new token and start working._

Welcome Observer pattern. So this should the workflow should look like

1.  If we have an invalid token, update a variable to represent that we are in the middle of getting new token.
2.  Make an actual request for the new token.
3.  Once the request is completed successfully, unset the variable to show that no request for auth is in progress.
4.  Send an event that new token is available so that other listeners can take appropriate action.

Step 1,2 and 3 are easy. We can maintain a global variable/state in Redis which request can check before making a new request.

Lets implement step 4: Lets start with our Observable code

{% gist 73691f53c80d21b4faaef513e056232e %}

Now we have to do it listen to `done` event and we are good. Something like this

{% gist 55772819baa52215f3d43c5066475b37 %}

Running the same test script would give the below output

<pre name="d5f2" id="d5f2" class="graf graf--pre graf-after--p">// betterServiceHandler
{"newToken":"def"}</pre>

<pre name="46b3" id="46b3" class="graf graf--pre graf-after--pre">// tokenGenerator
I am handling token: abc</pre>

As we can see, we only called the token generator service only once!

**UPDATE:**
As noted by [Hoseong Asher Lee](https://medium.com/@hoseong.a.lee), we should use `observable.once` instead of `observable.on` as mentioned in the blog. This would make sure an event handler is called only once