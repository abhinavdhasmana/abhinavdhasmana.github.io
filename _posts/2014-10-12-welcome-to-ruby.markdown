---
layout: post
title:  "Why ruby is awesome"
date:   2014-11-10 14:13:12
categories: Ruby
short_text: Why I love ruby
long_text: Why I love ruby
image: /images/i_love_ruby.jpg
author: Abhinav Dhasmana
---
Among many other things which I love about ruby, one thing that strikes me out is that its built for developers.

Other day, while working on the javascript, I wanted to find the unique elements inside an array. I just could not find an inbuilt method. I ended up writting this

{% highlight javascript %}
var unique = selected_values.filter(function(itm, i, selected_values) {
                return i == selected_values.indexOf(itm);
            });
{% endhighlight %}

In ruby, this is super simple. All I need to do is call a simple method called uniq.

{% highlight ruby %}
unique_elements = non_unique_elements.uniq
#=> Now unique_elements has all the unique elemets from the non_unique array.
{% endhighlight %}

If you want, you can actually modify the current array itself by using a bang operator.

{% highlight ruby %}
non_unique.uniq!
#=> Now non_unique has all the unique elemets from the non_unique array.
{% endhighlight %}

{% if true %}
  <div id="disqus_thread"></div>
  <script>
    var disqus_config = function () {
    this.page.url = "http://abhinavdhasmana.in/ruby/2014/11/10/welcome-to-ruby.html"; // Replace PAGE_URL with your page's canonical URL variable
    this.page.identifier = "ruby/2014/11/10/welcome-to-ruby.html";
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

<!-- Check out the [Jekyll docs][jekyll] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll’s dedicated Help repository][jekyll-help]. -->

[jekyll]:      http://jekyllrb.com
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-help]: https://github.com/jekyll/jekyll-help
