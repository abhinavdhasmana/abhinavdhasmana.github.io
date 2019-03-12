---
layout: post
title:  "Git 101 and its day to day usage: Part 1"
categories: [Git]
summary: Git commands
author: Abhinav Dhasmana
---

*This is a part one of a series of blog where I am planning to capture on how we provide technical apprenticeship to new college graduates that intern with us.*

First thing that any new software developer should learn is Source Control Management (SCM) and we use Git.

---
We try to use as much as open source as we can in our applications and this exercise is no different. For git we use [Git-it (Desktop App)]

Once we are done with the basics, there are few commands that we use which are not covered in the app but are essential in day to day operation.

## git stash

As the name suggest, this stashes the changes in a dirty working directory. This command is handy when you are working on something, and you have to switch your context to something else. While the same thing can be achieved if you create a branch but this is much faster and simpler (though the changes remain in your computer).

While git stash has many options, most commonly that are used are:

* git stash save <message>
* git stash pop stash@{<stash number>}
* git stash list
* git stash show stash@{<stash number>}

Lets take an example to understand how these commands can be effectively used.

* You are working on your develop branch and testing a fix for bug 137. Output of git diff is below

![Output of git diff](/images/blog/git101-1/git1_1.png){: .center-image }

* While working on this, a higher priority bug 168 comes along.
* You run git stash save WIP:FixForBug137. Output of git diff is empty.
* You start to work on fix for bug 168. Output of git diff is below

![Output of git diff](/images/blog/git101-1/git1_2.png){: .center-image }

* You are really having a bad day and you decided on take on a new bug. You stash these changes as well git stash save WIP:FixForBug168
* Output of git stash list will look like this

![Git stash list output](/images/blog/git101-1/git1_3.png){: .center-image }

* You can look at the content of each stash with git stash show -p stash@{1}. Output of the above command will be

![Output of git stash show](/images/blog/git101-1/git1_4.png){: .center-image }

You can apply any of these stashed and resume your work by simply using `git stash pop stash@{0}` or `git stash pop stash@{1}` . Output of git diff after running `git stash pop stash@{1}` is below

![Output of git diff after git stash pop](/images/blog/git101-1/git1_5.png){: .center-image }

`git stash` is branch independent by default. This means that if you rungit stash in one branch, move to another branch and then do git stash pop, the code is popped in that branch. **I find this as the easiest way to move my local code changes from one branch to another.**

PS: You can always run into conflicts when popping a stash. You can read here on how to handle it.

## git reset

Caution: **Use this command only on commits which are local and not that have been shared with anyone**
Consider that you have been working on a branch for few days with few files and you have decided to abandon this after validating the feasibility. Now you would like to go back to the initial state. You can use

* `git reset --hard <shaOfTheCommitWhereYouWantToGo>` to discard the changes in the working directory (dirty files which have not been committed) as well as files that have been committed.
* `git reset <shaOfTheCommitWhereYouWantToGo>` to make all the changes come to your working directory.

Lets look at the example:

Suppose this is the output of your git log and there are no local changes that are not committed.

![Output of git log](/images/blog/git101-1/git1_6.png){: .center-image }

As you can see, there is a spelling mistake, in the last commit. I wanted to say changes and not chantes. You can reset the head to the first commit (4f804b) and rewrite the commit message with the following commands.

`git reset4f804b06ac9617501bad9432d51eda65d6d48c4d`

With this, the git log and git diff looks like below images respectively.

![](/images/blog/git101-1/git1_7.png){: .center-image }

![](/images/blog/git101-1/git1_8.png){: .center-image }

Now I can correct my typing mistake and make the correct commit message with `git commit -am "Changes to file1`

However if you want to completely remove the changes in the second commit and permanently move to the previous commit, you can use `--hard` option.

So if you have two commits when this exercise was started and if you run `git reset --hard 4f804b06ac9617501bad9432d51eda65d6d48c4d`

The output of `git log` will be the same as with the without `--hard` example but the output of `git diff` would be empty and you would permanently loose the changes that you have made in the file1


[Git-it (Desktop App)]:https://github.com/jlord/git-it-electron