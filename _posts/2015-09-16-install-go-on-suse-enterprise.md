---
layout: post
title:  "How to install go in suse"
tags: golang
author: Abhinav Dhasmana
---

Task: Install go on a suse box.

Configuration: This is the output of `cat /etc/SuSE-release`
```
SUSE Linux Enterprise Server 11 (x86_64)
VERSION = 11
PATCHLEVEL = 3
```

This link `https://golang.org/dl/` do not provide any binaires for the Suse 11.


After a lot of searching, I came to this page `https://software.opensuse.org/download.html?project=devel%3Alanguages%3Ago&package=go`

and followed these two simple steps

`wget http://download.opensuse.org/repositories/devel:/languages:/go/SLE_11_SP3/x86_64/go-1.5-162.1.x86_64.rpm`

and `zypper install go-1.5-162.1.x86_64.rpm`

Done :)

{% if true %}
  <div id="disqus_thread"></div>
  <script>
    var disqus_config = function () {

    this.page.url = "http://abhinavdhasmana.in/golang/2015/09/16/install-go-on-suse-enterprise.html"; // Replace PAGE_URL with your page's canonical URL variable
    this.page.identifier = "golang/2015/09/16/install-go-on-suse-enterprise.html";
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