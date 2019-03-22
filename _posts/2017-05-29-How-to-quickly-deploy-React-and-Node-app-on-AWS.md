---
layout: post
title:  "How to quickly deploy React and Node app on AWS"
tags: [Node.js, React, AWS, DevOps]
summary: So you have a setup running on your local box with node server and a webpack server and want to deploy this code on a brand new aws ubuntu box.
author: Abhinav Dhasmana
---

So you have a setup running on your local box with node server and a webpack server and want to deploy this code on a brand new aws ubuntu box. The expectation is when you visit your public ip address, your home route should work and your react landing page should be served. For this to happen, we need to do these things.

*   Enable your node code to serve react code.
*   Get your code from github/gitlab etc on this new box.
*   Map port 80 to the port your node server is running.

* * *

**Enable your node code to serve react code:** Before we hand over react code to node, lets create a production build for our react code. If you are using [create-react-app](https://github.com/facebookincubator/create-react-app), running `npm run build` will do the trick. Now copy the `build` folder in the public folder of your node app and commit these changes.

Second part is to enable the node app to serve static content. If you are using hapi, you can follow [this](https://hapijs.com/tutorials/serving-files), which is few lines of code to serve the static content.

You can test this by running this on your localbox and going to the port on which your node server is running (for eg: [http://localhost:3000)](http://localhost:3000%29)

* * *

**Get your code from github/gitlab etc on this new box:**

*   Install git `apt-get install git`
*   Install nvm

<pre name="b066" id="b066" class="graf graf--pre graf-after--li">curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash</pre>

*   Install node `nvm install 7.2.0` // change it to the version you want
*   Install database. I am using postgres .

<pre name="6961" id="6961" class="graf graf--pre graf-after--li"> apt-get install postgresql postgresql-contrib</pre>

To connect to the database, make sure your configuration files have the right username and password. You might also have to create the database manually on which you want to connect your app.

*   Now we would clone your code. You can use https to clone but it is painful to do it every time. Lets generate public/private keys which you can enter into our repo settings so that we can clone easily without the password.
*   Create keys:

<pre name="a403" id="a403" class="graf graf--pre graf-after--li">ssh-keygen -t rsa -b 4096 -C "_your_email@example.com_"</pre>

Now copy the contents of your public key `cat ~/.ssh/id_rsa.pub`into your github/gitlab repo.

Now in your aws box, run `git clone git@<your repo>`and this will clone the repo in your box. Run `npm install` and `npm start`or whatever command you have in your `package.json` and we have our app up and running.

* * *

**Map port 80 to the port your node server is running:** If we would type your aws machine ip address into the box, nothing would work as your app is running on port 3000 and not on port 80\. To enable this, we can run this command

<pre name="2a38" id="2a38" class="graf graf--pre graf-after--p">sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-ports 3000</pre>

What this command is doing is appending (flag `-A` ) the `iptables PREROUTING` chain and saying that any `tcp`protocol with a destination port (flag `— dport`) of 80 should be redirected to port 3000.

P.S: Every time a server re-start, this setting go away as they are stored in memory. You can persist these rules as well. Look [here](https://serversforhackers.com/video/firewall-persisting-iptables-rules) for a tutorial.

P.P.S: We could have used nginx to run on port 80 and map to our server port. It would have been a little more involved though.

Once done, you are good to go!!

PS: If you are looking to automate all these steps, I have written a new blog post: [How to deploy React and Node app on AWS: A better approach]({% post_url 2018-05-15-how-to-deploy-react-and-node-app-on-aws-a-better-approach %})