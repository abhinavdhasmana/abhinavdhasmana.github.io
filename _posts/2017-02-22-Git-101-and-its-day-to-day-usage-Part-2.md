---
layout: post
title:  "Git 101 and its day to day usage: Part 2"
tag: [Git]
summary: Git commands
author: Abhinav Dhasmana
---

_This is a part two of a series of blog where I am planning to capture on how we provide technical apprenticeship to new college graduates that intern with us. Part 1 can be found_ [_here_]({% post_url 2017-02-15-git-101-and-its-day-to-day-usage-part-1 %})

* * *

If you noticed in [part 1]({% post_url 2017-02-15-git-101-and-its-day-to-day-usage-part-1 %}), everything was about working with git changes that are local on your box. This post will cover some common scenarios which you might encounter.

**Scenario 1:** You are starting a new feature/bug fix.

Make sure your master is updated. This can avoid conflicts and make merging easier.

Simply do a `git pull`.

Initial state

![](/images/blog/git101-2/git2_1.png){: .center-image }

After git pull

![](/images/blog/git101-2/git2_2.png){: .center-image }

Now, you can create a new branch from master

**Scenario 2: You have completed your work on your branch and want to make this change to master.**

You can run `git merge feature-branch` on a master branch

There can be two scenarios here:

**2.1** When no commit has been made to master since you created your own branch.

![](/images/blog/git101-2/git2_3.png){: .center-image }

After merge, the git tree would look like

![](/images/blog/git101-2/git2_4.png){: .center-image }

This merge is known as fast forward merge as all git had to do was to move its head from 2 to 4.

2.2 When commit has been made to master since you created your own branch.

![](/images/blog/git101-2/git2_5.png){: .center-image }

In a scenario like this, git creates a new commit, a merge commit, and points your master to it. This is known as 3-way merge

![](/images/blog/git101-2/git2_6.png){: .center-image }

In the above diagram, commit 6 is created by git. In this scenario, we can enter into a situation where we have a conflict. You can get a message like

```
CONFLICT (content): Merge conflict in app/index.js
Automatic merge failed; fix conflicts and then commit the result.
```

To solve this, you would have to do the following steps.

```
1\. Manually remove conflict from index.js
2\. git add app/index.js
3\. git commit -m '<your commit message>'</pre>
```

**Scenario 3: There is more than one person working on your branch. You want to incorporate those changes before raising a pull request**

This scenario is no different than scenario 1\. Just run `git pull origin feature-branch`and you are good.

**Scenario 4: You want to incorporate changes from another branch**

There are scenarios where you would like to take a change from another branch on which you would like to develop upon.

![](/images/blog/git101-2/git2_7.png){: .center-image }

Now, you want changes in `commit 4` and build on top of it. You can run the following commands to make this happen

```
1\. git checkout master
2\. git pull origin master
3\. git checkout feature-branch
4\. git rebase master</pre>
```

After running this, your git tree will look like this.

![](/images/blog/git101-2/git2_8.png){: .center-image }

Note that the commit after 4 are `3'` and `5'` and not `3` and `5` as rebase rewrites the commit history.

Missed anything? Let me know in comments and I’ll add it.