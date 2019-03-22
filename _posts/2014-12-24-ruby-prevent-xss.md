---
layout: post
title:  "How to prevent XSS attack in Rails Controller"
tags: Ruby
author: Abhinav Dhasmana
---

There is good amount of documentation avaiable on what Cross-site Scripting(XSS)] and best practices [here][ror-security] and [here][owsap]

I wanted to make sure that even if someone modifies the content mid way between the form submission and server, my controller should be able to take care of this.

In order to solve this, I used the rails [sanitize][sanitize] ActionView::Helper method to solve this issue.

Added ActionView::Helpers::TextHelper in the controller

{% highlight ruby %}

class MyController < ApplicationController
  include ActionView::Helpers::TextHelper

{% endhighlight %}

and when parsing the params, we can simple add a new key to the params and use that

{% highlight ruby %}

params[:sanitized_text] = []
params[:text].each_with_index do |ins|
  params[:sanitized_text] << sanitize(ins)
end
Object.rebuild_object(params[:account_id], params[:sanitized_text])

{% endhighlight %}

{% if true %}
  <div id="disqus_thread"></div>
  <script>
    var disqus_config = function () {
    this.page.url = "http://www.abhinavdhasmana.in/ruby/2014/12/24/ruby-prevent-xss.html"; // Replace PAGE_URL with your page's canonical URL variable
    this.page.identifier = "/ruby/2014/12/24/ruby-prevent-xss.html";
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

[ror-security]:      http://guides.rubyonrails.org/security.html#cross-site-scripting-xss
[owsap]:             https://www.owasp.org/index.php/Ruby_on_Rails_Cheatsheet#Cross-site_Scripting_.28XSS.29
[sanitize]: http://api.rubyonrails.org/classes/ActionView/Helpers/SanitizeHelper.html
