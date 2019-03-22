---
layout: post
title:  "Effectively communicate between Microservices"
tags: [Node.js, Golang, grpc]
summary: "Choose between http/1.1 and gRPC. TL; DR: Use gRPC"
author: Abhinav Dhasmana
---

How microservices communicate with each other can affect performance and scalability of the application. Communication between services can be synchronous or asynchronous. For this blog post, we will focus on synchronous. We have 2 common protocols at our disposal

*   **HTTP/1.1**: The “default” http call
*   **gRPC:** High performance remote procedure call (RPC framework). [_gRPC.io_](https://grpc.io/)has a comprehensive documentation and how it works.

Let’s look at some sample code and run some performance test to pick our choices.

### Non-cluster mode

Let’s start small and try to get one microservice talk to another one.

![](/images/blog/grpc/1.png){: .center-image }

Our application has following components

*   **Load Testing tool:** [jMeter](https://jmeter.apache.org/)
*   **Service A:** Makes a request to serviceB and returns that response.
*   **Service B:** Replies with a static JSON after 10ms of delay for all the APIs
*   **VMs:** Both the VMs are Amazon EC2 t2.xlarge machines

### HTTP/1.1

This is the default request we make when we use any of the HTTP client libraries like [axios](https://github.com/axios/axios), [superagent](https://github.com/visionmedia/superagent).

Let’s create `ServiceB` to implement our APIs.

{% gist 359c7a4b723d945f2e93235777a8423e %}

Next, we create `ServiceA` which calls `ServiceB` with HTTP/1.1

{% gist 25c44ba03256a4dbb50e7e28aa40224b %}

With these two services running, we can now fire-up our `jMeter` to run some performance test. We will use 50 users will 2000 request each. As we can see in the screenshot below, our median is `37ms` .

![](/images/blog/grpc/2.png){: .center-image }

### gRPC

gRPC uses [protocol buffers](https://developers.google.com/protocol-buffers/docs/overview) by default. In addition to `serviceA` and `serviceB`, we also need a `proto` file which will define our remote calls.

{% gist a39cfe640f82d3792f757210f6808dd0 %}

Let’s implement `serviceB` again, this time using `gRPC`

{% gist 5229a2adc41e8fdb10a69ac76515b831 %}

Few things to note:

*   We create `grpc` server at `line 4`
*   We add a service to our server at `line 16`
*   We bind `port` and `credentials` at line 23

`serviceA` implementation using gRPC is below

{% gist f9769726c19d08c3d8b38964be757324 %}

Few things to note:

*   We create `grpc` client at `line 4`
*   We make a remote call at `line 6`

Let’s run our jMeter tool test again.

![](/images/blog/grpc/3.png){: .center-image }

As we can see from this data, `gRPC` is 27% faster than the normal HTTP/1.1 request

### Application architecture in cluster mode

![](/images/blog/grpc/4.png){: .center-image }

If you are running anything in production, it is most likely you are running many instances of any given service. In this case, we are making the following changes:

*   Running three instances of our service on different ports
*   We are using [NGINX](https://www.nginx.com/) as a load balancer

### HTTP/1.1

No changes are required on the services side. All we need to do it configure our `nginx` to load balance the traffic with `serviceB` . We can modify the `/etc/nginx/sites-available/default` to look like this

{% gist ed97a8356329c9160b834b83eced1b61 %}


We can now start our 3 instances of `serviceB` and run our performance test

![](/images/blog/grpc/5.png){: .center-image }

### gRPC

Support for gRPC was recently added to [nginx version 1..13.10](https://www.nginx.com/blog/nginx-1-13-10-grpc/) . So, we need to get the latest one as the default `sudo apt-get install nginx` will not work.

We are not using node in [cluster](https://nodejs.org/api/cluster.html) mode as gRPC is [not supported](https://github.com/grpc/grpc/issues/6976).

Follow the steps below to get the latest `nginx`

{% gist 247d92b663677f9e5eb7a0ee0990fa83 %}

We would also need the ssl certificates. We can create self-signed certificate using [openSSL](https://www.openssl.org/)

{% gist 13a861615bd14b2bc20a7b45aec52ab5 %}

Next, we need to change our `/etc/nginx/sites-available/default` file to start using `gRPC`.

{% gist be35cc88156350e7498e0a940ffd477b %}

That’s it. Let’s run our performance test again

![](/images/blog/grpc/6.png){: .center-image }

As we can see `gRPC` are 31% faster in comparison to the HTTP/1.1

### Conclusion

gRPC is fast. In the tests above, we have **31%** gain over HTTP/1.1 by just changing on how we talk to each other.

The full source code can be found on [GitHub](https://github.com/abhinavdhasmana/interserviceCommunication)