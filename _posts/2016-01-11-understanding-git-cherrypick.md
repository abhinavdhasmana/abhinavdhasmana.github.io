---
layout: post
title:  "Understand git cherry pick"
tag: git
image: /images/blog/git-cherry-pick/git-cherry-pick.png
author: Abhinav Dhasmana
---
Simply put, `git cherry pick` is used to take the contents of one commit and put it on top of your current branch.

The command is pretty simple

```
git cherry-pick <SHA>
```

where SHA is the SHA of the commit you want to take. [Here][git-cherry-pick-documentation] is the official documentation.

What the documentation does not mention is what gets picked when you take that commit. Is it the diff in the commit or is it the whole file at that point that comes in? I did a small experiement to see what happens. The whole source code can be found on [Github][github-cherry-pick]

* Create two commits on the master branch which contains 2 simple text files.
* Create a new branch `cherrypick`.
* Create a new commit. This commit modifies one of the existing file (2.txt) and adds a new file (3.txt).

<img src="/images/blog/git-cherry-pick/diff1.png" alt="Commit 1" style="width: 720px;"/>


* Make another commit on cherrypick branch. This again modifies the 2.txt file again.

<img src="/images/blog/git-cherry-pick/diff2.png" alt="Commit 2" style="width: 720px;"/>


Now I need a change that is made in step 4. So this is what I did

```
1. checkout the master branch.
2. git cherry-pick 03c6f6d6977a483c78633083ce94c2ccaaf721c6 (sha of the commit on the cherrypick branch)
```

As expected, I got a conflict in 2.txt as expected. What I got in conflict was not the diff in the second commit but for the whole file.

```
This is the first line in the second file
<<<<<<<
This is the second line in the second file
=======
This is the second line in the second file
This is the first line in the second file for the new branch
This is the second line in the second file for the new branch
>>>>>>>
```

Conclusion: When we do cherry-pick, all the contents of that file will come and not just the diff in that commit. Remember that git is not diff based, it creates the whole snapshot with [every commit][git-as-snapshot]

{% if true %}
  <div id="disqus_thread"></div>
  <script>
    var disqus_config = function () {

    this.page.url = "http://www.abhinavdhasmana.in/git/2016/01/11/understanding-git-cherrypick.html"; // Replace PAGE_URL with your page's canonical URL variable
    this.page.identifier = "git/2016/01/11/understanding-git-cherrypick.html";
    };

    (function() { // DON'T EDIT BELOW THIS LINE
      var d = document, s = d.createElement('script');
      s.src = '//abhinavdhasmana.disqus.com/embed.js';
      s.setAttribute('data-timestamp', +new Date());
      (d.head || d.body).appendChild(s);
      })();
  </script>
  <noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
{% endif %}

[git-cherry-pick-documentation]: https://git-scm.com/docs/git-cherry-pick
[github-cherry-pick]: https://github.com/abhinavdhasmana/cherry-pick-example
[git-as-snapshot]: https://git-scm.com/book/en/v2/Getting-Started-Git-Basics