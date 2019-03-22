---
layout: post
title:  "How to deploy Node.js app on AWS with GitLab"
tags: [Node.js, AWS, Gitlab, Continuous Integration]
author: Abhinav Dhasmana
---

<iframe width="560" height="315" src="https://www.youtube.com/embed/BKzddvR6UMg" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>


We are going to achieve following things in this article:

*   Have GitLab run `lint` and `test` on every branch of our code (Continuous Integration)
*   If everything passes in previous step, tell GitLab to deploy these changes to AWS EC2 instances only for **master branch**(Continuous Deployment)

* * *

### Continuous Integration with GitLab

[GitLab has excellent support for CI](https://about.gitlab.com/features/gitlab-ci-cd/). To enable this, we need to create `.gitlab-ci.yml` file.

{% gist cb352a02eec361354d7eeef04bb829ec %}

Lets understand this file.

_Line 2:_ Tells which Docker image to use

_Line 5,6:_ Run npm install before any job

_Line 8–9:_ We have created only 1 stage with a name `test` , could be any name

_Line 15–18:_ We have created a job with a name `lint` . Again this could be any name. We want this job to run in a test stage. We want it to run `npm run lint` command. `lint` script is defined in our `package.json` file.

_Line 20–23:_ Same as previous job but this time, run `npm run test` command

Since we have not provided any branch, this would run on all the branches whenever a commit is made. Important thing to note is that all the jobs ( `lint` and `test`) will run in parallel as they are part of the same stage. If you want them to run sequentially, you can write multiple command inside `script` tag.

![](/images/blog/gitlab/gitlab1.png){: .center-image }

[Official documentation](https://docs.gitlab.com/ee/ci/yaml/) has more details about the yml config file.

Once we commit this file, GitLab would start running your script and if everything passes, we would see something like this inside our pipeline.

* * *

### **Continuous Deployment with GitLab**

We need to do three steps for deployment

*   Update our GitLab config to prepare our docker image
*   Enable GitLab container to ssh into our remote servers
*   Take the latest changes from GitLab and restart server

### Update .gitlab-ci.yml

Lets update our `.gitlab-ci.yml` file to reflect our new requirement

{% gist 522812edbe9a331a23ad521e4db4c27e %}

We have made three changes in the above file.

*   _Line 6:_ We want to make sure our instance has `openssh-client` installed. More on this later.
*   _Line 11:_ We have added another stage called `deploy` . It will only run after successful completion of `test` stage.
*   _Line 29–34:_ We would like to run this on master branch only. We want it to run a script located at `deploy/deploy.sh`

### Enable GitLab container to ssh into AWS EC2 instance

We need to solve for two problems:

*   **_How to give_** `**_ssh_**` **_key and server information to GitLab docker container so that it can ssh into EC2 instance:_**If we want to ssh into our AWS EC2 instance from our local machine, we do something like this

<pre name="2755" id="2755" class="graf graf--pre graf-after--li">ssh -i myPemFile.pem ubuntu@ {ip address of the instance}</pre>

It’s not a good idea to commit the key in our source code even if its private repo. So we will keep it in GitLab as an environment variable. Go to “Settings => CI/CD” and create two variables:

![](/images/blog/gitlab/gitlab2.png){: .center-image }

```
PRIVATE_KEY <Insert the key you downloaded when creating an EC2 instance>
```

```
DEPLOY_SERVERS <comma separated ip values of all the servers to which you want to deploy this code>
```

These variables would be available to us in our docker container as environment variables. We installed `ssh-agent` in our GitLab docker container so that we can ssh into the EC2 instance. Lets start the ssh agent and add this key to this docker image.

<pre name="5b6e" id="5b6e" class="graf graf--pre graf-after--p">eval $(ssh-agent -s)
echo "$PRIVATE_KEY" | tr -d '\r' | ssh-add - > /dev/null</pre>

GitLab at times add spaces which will fail `ssh` into the EC2 instance and hence the `tr -d ‘\r'` . Issue [here](https://gitlab.com/gitlab-examples/ssh-private-key/issues/1).

*   **_How to disable the prompt for_** `**_hostKeyChecking:_**`Whenever we ssh into a EC2 instance that we have never sshed before, we get a prompt like this

![](/images/blog/gitlab/gitlab3.png){: .center-image }host key checking when we ssh into the instance for the first time

We need to disable this as it would stop our deployment.

To disable this, we need to create a entry in ~`/.ssh/config` to looking something like this

```
Host *    StrictHostKeyChecking no
```

{% gist 375237f8b1fb7e5e815b06ed82dd9422 %}

Make sure this file is executable. We can change the permission with `chmod a+x` during commit and during our deploy script.

Now that we have solved both the problems, all we need is a bash script which can loop through the all the ips, ssh into each one of them and execute the script

<pre name="b880" id="b880" class="graf graf--pre graf-after--p">DEPLOY_SERVERS=$DEPLOY_SERVERS // We defined this in GitLab
ALL_SERVERS=(${DEPLOY_SERVERS//,/ })
for server in "${ALL_SERVERS[@]}"
do
  ssh ubuntu@${server} 'bash' < ./deploy/updateAndRestart.sh
done</pre>

So our full `deploy.sh` would look like this

{%gist 43477ad21955356318b59dc456059e99%}

### Take the latest changes from GitLab and restart server

This is what we want after we ssh into our box.

*   **Delete our own repository and clone again**: Since we are not using any tool like capistrano etc to manage our deployment, a reliable way to do this is to delete the old code and clone again. If you have a better solution than deleting the code, please leave it in comments.
*   **npm install**:The catch here is when we do non-interactive ssh into an instance. This menas that `~/.bashrc` is not loaded which in turn load our `nvm.sh` file which loads our node. So lets load it explicitly. Here is a great article on [Configuring your login sessions with dot files](http://mywiki.wooledge.org/DotFiles)
*   **restart server**: Next problem we need to tackle is pm2 restart. We cannot run pm2 from the source folder as we are deleting this folder and cloning again. PM2 refer the old folder which is not existent and will give error. This is the reason we do not start pm2 daemon from the source repo.

{% gist f8ee5b25b51fee4b1f0fa7e9f4623761 %}
That’s it!! Now every time a commit/merge is made into the master branch, it would be deployed to all the servers.

The full code is available [here](https://gitlab.com/abhinavdhasmana/ci_cd_demo)

Happy coding!!