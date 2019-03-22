---
layout: post
title:  "My first attempt to contribute to rails"
tags: opensource
author: Abhinav Dhasmana
---
After pondering over months, I decided to try and contribute to rails. I setup by system as described in the [Contributing to Ruby on Rails][Contributing-to-Ruby-on-Rails]

After setting up my system and few more days of figuring out where to start, I decided to look at the recent pull requests that were merged into the rails code and see if I can add any value there.

After looking at few commits, I saw this commit

```
'https://github.com/rails/rails/commit/d88f6e79ccf8480f349e985d7418d86b8f68cdda'
```

Its a nice request where the code has been moved to its each individual function. I thought that I can make this call even more elegant. So I rewrote the code from

{% highlight ruby %}

def type_to_sql(type, limit = nil, precision = nil, scale = nil)
  case type.to_s
  when 'binary'
    binary_to_sql(limit)
  when 'integer'
    integer_to_sql(limit)
  when 'text'
    text_to_sql(limit)
  else
    super
  end
end
{% endhighlight %}

to

{% highlight ruby %}

def type_to_sql(type, limit = nil, precision = nil, scale = nil)
  if (["binary", "integer", "text"].include?(type.to_s))
    send("#{type.to_s}_to_sql", limit)
  else
    super
  end
end

{% endhighlight %}


I thought this was a good enough code change for a push

  1. Reducing lines of code from 12 to 7

  2. If this array ```["binary", "integer", "text"]``` was shared between differetn databases say postgres and mysql, we could have move this to a yml file.

  3. Reducing lines of code from 12 to 7

However it was [rejected][rejected-pull-request] with the following comments

```
Thank you for the pull request but this just make harder to understand the code. The conditional is still there
```

After it was rejected, I tried to run perfomance test and see how does the system perform between the new code and the old code.

{% highlight ruby %}
  require 'benchmark/ips'

  def original_code(type, limit = nil)
    case type.to_s
    when 'binary'
      binary_to_sql(limit)
    when 'integer'
      integer_to_sql(limit)
    when 'text'
      text_to_sql(limit)
    else
      call_super
    end
  end

  def binary_to_sql(limit)
    puts "I am binary"
  end

  def integer_to_sql(limit)
    puts "I am integer"
  end

  def text_to_sql(limit)
    puts "I am text"
  end

  def call_super
    puts "I am super"
  end

  def new_code(type, limit = nil)
    if (["binary", "integer", "text"].include?(type.to_s))
      send("#{type.to_s}_to_sql", limit)
    else
      call_super
    end

  end

  Benchmark.ips do |x|
    x.report('original_code_with_garbage') {original_code('garbage')}
    x.report('new_code_with_garbage') {new_code('garbage')}
    x.compare!
  end

{% endhighlight %}

I started with the default to see the performance. The results were not encouraging

{% highlight ruby %}
Comparison:
original_code_with_garbage:   165024.7 i/s
new_code_with_garbage:   153540.2 i/s - 1.07x slower
{% endhighlight %}

As I moved further, things got even worse

{% highlight ruby %}
Comparison:
original_code_with_binary:   153336.1 i/s
new_code_with_binary:   137471.6 i/s - 1.12x slower


Comparison:
original_code_with_integer:   162532.3 i/s
new_code_with_integer:   133223.2 i/s - 1.22x slower

Comparison:
original_code_with_text:   162927.4 i/s
new_code_with_text:   128703.8 i/s - 1.27x slower

{% endhighlight %}

Of course, My pull request did not deserve to be accepted.

My learning from this:
* Since searching in an array is linear time, as we moved away from the start of an array, the perfomance descreased.
* Always do benchmark test before submitting the request. I do not want to make rails slower even bu 0.001%
* Smaller code footprint is not always the best solution.

{% if true %}
  <div id="disqus_thread"></div>
  <script>
    var disqus_config = function () {

    this.page.url = "http://abhinavdhasmana.in/opensource/2015/05/18/First-failed-attempt-to-become-a-rails-contributor.html"; // Replace PAGE_URL with your page's canonical URL variable
    this.page.identifier = "opensource/2015/05/18/First-failed-attempt-to-become-a-rails-contributor.html";
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






[Contributing-to-Ruby-on-Rails]: http://guides.rubyonrails.org/contributing_to_ruby_on_rails.html

[rejected-pull-request]: https://github.com/rails/rails/pull/20162


