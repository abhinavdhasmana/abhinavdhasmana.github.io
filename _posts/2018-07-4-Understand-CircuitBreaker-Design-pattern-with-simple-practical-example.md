---
layout: post
title:  "Understand CircuitBreaker Design pattern with simple practical example"
categories: [Node.js, Design Pattern, JavaScript]
author: Abhinav Dhasmana
---

**Design pattern for “when shit happens”**

<br />
<br />

### Problem Statement

We have a `serviceA` which has two APIs

*   `/data` which depends on `serviceB`
*   `/data2` does not depend on any external service

![](/images/blog/circuit-breaker/1.png){: .center-image }

Let’s try and implement this scenario and see how it affects our whole system. The full source can be found on [Github](https://github.com/abhinavdhasmana/circuitBreaker).

### Without Circuit Breaker

`serviceB` implementation below. The API is returning a 5 second delayed response to a request for the first 5 minutes. It’s running on port 8000.

{% gist de84b72df41341b226209a14fc762db9 %}

`serviceA` implementation which will make http request to `serviceB`

{% gist 02e3d7affb31ff9b4c01bb97e6ba894c %}

We will simulate the load using `[jMeter](https://jmeter.apache.org/)` . Within few seconds, `serviceA` would be starved of resources. All the requests are waiting for http request to complete. First API would start throwing error and it will eventually crash as it would reach its max heap size.

![](/images/blog/circuit-breaker/2.png){: .center-image }

<pre name="6cbd" id="6cbd" class="graf graf--pre graf-after--figure"><--- Last few GCs --->

[90303:0x102801600]    90966 ms: Mark-sweep 1411.7 (1463.4) -> 1411.3 (1447.4) MB, 1388.3 / 0.0 ms  (+ 0.0 ms in 0 steps since start of marking, biggest step 0.0 ms, walltime since start of marking 1388 ms) last resort GC in old space requested
[90303:0x102801600]    92377 ms: Mark-sweep 1411.3 (1447.4) -> 1411.7 (1447.4) MB, 1410.9 / 0.0 ms  last resort GC in old space requested

<--- JS stacktrace --->

==== JS stack trace =========================================

Security context: 0x2c271c925ee1 {JSObject}
    1: clone [/Users/abhinavdhasmana/Documents/Personal/sourcecode/circuitBreaker/client/node_modules/hoek/lib/index.js:~20] [pc=0x10ea64e3ebcb](this=0x2c2775156bd9 <Object map = 0x2c276089fe19>,obj=0x2c277be1e761 {WritableState map = 0x2c27608b1329>,seen=0x2c2791b76f41 <Map map = 0x2c272c2848d9>)
    2: clone [/Users/abhinavdhasmana//circuitBreaker/client/node_modul...
</pre>

Now, instead of one, we have two services which are not working. This would escalate throughout the system and the whole infrastructure will come down.

### Why we need a circuit breaker

In case we have `serviceB` down, `serviceA` should still try to recover from this and try to do one of the followings:

*   **Custom fallback:** Try to get the same data from some other source. If not possible, use its own cache value.
*   **Fail fast:** If `serviceA` knows that `serviceB` is down, there is no point waiting for the timeout and consuming its own resources. It should return ASAP “knowing” that `serviceB` is down
*   **Don’t crash:** As we saw in this case, `serviceA` should not have crashed.
*   **Heal automatic:** Periodically check if `serviceB` is working again.
*   **Other APIs should work:** All other APIs should continue to work.

### What is circuit breaker design?

The idea behind is simple:

*   Once `serviceA` “knows” that `serviceB` is down, there is no need to make request to `serviceB`. `serviceA` should return cached data or timeout error as soon as it can. This is the **OPEN** state of the circuit
*   Once `serviceA` “knows” that `serviceB` is up, we can **CLOSE** the circuit so that request can be made to `serviceB` again.
*   Periodically make fresh calls to `serviceB` to see if it is successfully returning the result. This state is **HALF-OPEN**.

![](/images/blog/circuit-breaker/3.png){: .center-image }

This is how our circuit state diagram would look like

![](/images/blog/circuit-breaker/4.png){: .center-image }

### Implementation with Circuit Breaker

Let’s implement a `circuitBreaker` which makes GET http calls. We need three parameters for our simple `circuitBreaker`

*   How many failures should happen before we OPEN the circuit.
*   What is the time period after which we should retry the failed service once the circuit is in OPEN state?
*   In our case, the timeout for the API request.

With this information, we can create our `circuitBreaker` class.

{% gist f54be690482bc5fe1e2da0f93eda6ebd %}

Next, let’s implement a function which would call the API to `serviceB` .

{% gist bca3e5bbc13e65ff7ae62dbc7fb88f30 %}

Let’s implement all the associated functions.

{% gist dd942e34749cd7d3e9246820a030f1f8 %}

Next step is to modify our `serviceA` . We would wrap our call inside the `circuitBreaker` we just created.

{% gist f172caec5217123bdb20972c730a9d7b %}

Important changes to note in this code with respect to the previous code:

*   We are initializing the circuitBreaker `const circuitBreaker = new CircuitBreaker(3000, 5, 2000);`
*   We are calling the API via our circuit breaker `const response = await circuitBreaker.call(‘http://0.0.0.0:8000/flakycall');`

That’s it! Now let’s run our jMeter test again and we can see that our `serviceA` is not crashing and our error rate has gone down significantly.

![](/images/blog/circuit-breaker/5.png){: .center-image }

### Further reading

*   Martin Fowler on [CircuitBreaker](https://martinfowler.com/bliki/CircuitBreaker.html)
*   Netflix: [Making the Netflix API More Resilient](https://medium.com/netflix-techblog/making-the-netflix-api-more-resilient-a8ec62159c2d)
*   Netflix: [Fault Tolerance in a High Volume, Distributed System](https://medium.com/netflix-techblog/fault-tolerance-in-a-high-volume-distributed-system-91ab4faae74a)