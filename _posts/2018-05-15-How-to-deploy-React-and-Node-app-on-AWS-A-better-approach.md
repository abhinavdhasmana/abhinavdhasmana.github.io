---
layout: post
title:  "How to deploy React and Node app on AWS: A better approach"
tags: [Node.js, AWS, Gitlab, Nginx]
author: Abhinav Dhasmana
---

In one of my previous blog, [_How to quickly deploy React and Node app on AWS_]({% post_url 2017-05-29-how-to-quickly-deploy-react-and-node-app-on-aws %})_,_ I explained how we can _quickly_ deploy our react app, node app and enable them to talk to each other. Essentially, it involved the following steps:

*   Make a production build of your react app
*   Put these files manually in the public folder of your node server
*   Enable node.js to serve static files
*   Setup node.js, postgres on the aws EC2 instance
*   forward request to port 80 to node.js server

The above workflow works if you do not have to deploy regularly. We have to perform following steps **manually** every-time a change happens in the react app

*   Building react app
*   Copy react files to node.js public folder
*   Deploy the node.js app

Let’s look at an alternative design and automate the whole deployment process.

* * *

### New architecture

In this design, we would decouple our Node app and React app. We would introduce `nginx` to facilitate communication between react and node.

![](/images/blog/deploy-gitlab/gitlab1.png){: .center-image } Use of nginx to serve react code and act as a reverse proxy for the node server

* * *

### **New Workflow**

![](/images/blog/deploy-gitlab/gitlab2.png){: .center-image }
![](/images/blog/deploy-gitlab/gitlab3.png){: .center-image }

* * *

### **Deploy Node App**

This is covered in my blog post: [How to deploy Node.js app on AWS with GitLab]{% post_url 2018-05-03-how-to-deploy-node-js-app-on-aws-with-gitlab %}
### Deploy React App

This is very similar, rather much simpler than node deployment, as we do not have to ssh into the AWS EC2 instance.

As with node app deployment, we need to setup our `.gitlab-ci.yml` file. This file would exactly be the same as that of the node app except one change. We need to build our react app with `npm run build` at line 29

{% gist 37ee57df91208518c852d6cce2faf6a8 %} gitlab config for the react app

Now that we have the build of our react app, we need to move these files to a folder where `nginx` can read from. All the things would be similar to the node deployment except we would copy the files with `scp` instead of `ssh` at line 34

{% gist 73fce6ac5159f095f2b50289dda64032 %} Deploy script for react app on AWS

### Configure Nginx

Now that we have code of both Node and React on our AWS box, we need a means through which they can talk to each other.

Let’s install Nginx on our Ubuntu box

`sudo apt-get install nginx`

Next, edit the location of the default page that Nginx will serve.

Edit this file `/etc/nginx/sites-available/default` , and add the following line of code inside the `server` block

`location / {
 root /home/ubuntu/myFrontApp/dist;
 index index.html index.htm index.nginx-debian.html;
 }`

So our default page would be now `index.html` at `/home/ubuntu/myFrontApp/dist` location.

Suppose that react app makes a request to the node server with `api` path. Something like `http://<ip>/api/user` to get the list of all users. So we need to make sure that every route with `/api` should be redirected to node server.

In the `/etc/nginx/sites-available/default` file, we will add the following lines

`location /api/ {
 proxy_pass [http://localhost:8000/](http://localhost:8000/);
 proxy_http_version 1.1;
 proxy_set_header Upgrade $http_upgrade;
 proxy_set_header Connection ‘upgrade’;
 proxy_set_header Host $host;
 proxy_cache_bypass $http_upgrade;
 }`

So the whole block looks like below in the nginx file

<pre name="77c1" id="77c1" class="graf graf--pre graf-after--p">server {
 listen 80 default_server;
 listen [::]:80 default_server;
 location / {
   root /home/ubuntu/hssfrontend/dist;
   index index.html index.htm index.nginx-debian.html;
        }
 server_name _;</pre>

<pre name="34a5" id="34a5" class="graf graf--pre graf-after--pre">location /api/ {
    proxy_pass [http://localhost:8000/](http://localhost:8000/);
    proxy_http_version 1.1;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection 'upgrade';
    proxy_set_header Host $host;
    proxy_cache_bypass $http_upgrade;
 }
}</pre>

Save this file and restart nginx server with `sudo service nginx restart` .

Once done, you are good to go!!