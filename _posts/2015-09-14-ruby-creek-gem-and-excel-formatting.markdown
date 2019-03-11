---
layout: post
title:  "Creek gem and how it handles cell formatting"
categories: ruby
author: Abhinav Dhasmana
---

[Creek][creek-github] is an awesome gem that I have been using to parse the large excel files. It parses the xml contents of the excel and returns the data as a hash.

However in one of the scenario, I saw that while the excel showed the value of `42.7068493150685` but the database(mysql) was storing the value of 42. First reaction was to change the datatype from float to double. As expected that did not help.

Next I opened up `rails c` and loaded the same object. Manually assigned the value the attribute and it saved correctly.

This brought me to inspect on how creek is returning data. This is what I experimented.

Created two columns in a simple excel. First column (`Data 1`) will always be General while the second column(`Data 2`) I'll keep on changing.

First one is pretty simple, both `Data 1` and `Data 2` are formatted as general as shown below
![below](/img/blog/creek_excel/creek_general.png)

I run the following code.

{% highlight ruby %}


gem 'creek', '~>1.0.8'
gem 'awesome_print'

require 'creek'
require 'awesome_print'
creek = Creek::Book.new ("datafile/excel_with_general_formatting.xlsx")
sheet = creek.sheets[0]
ap puts sheet.rows.to_a[1]

{% endhighlight %}

This results in the following output

```{"A2"=>"42.706849315068503", "B2"=>"42.706849315068503"}```

which makes sense.


Next time I changed `Data 2` to Number with 2 decimal precision.

![](/img/blog/creek_excel/creek_floating.png)



I run the following code.

{% highlight ruby %}


gem 'creek', '~>1.0.8'
gem 'awesome_print'

require 'creek'
require 'awesome_print'
creek = Creek::Book.new ("datafile/excel_with_floating_formatting.xlsx")
sheet = creek.sheets[0]
ap puts sheet.rows.to_a[1]

{% endhighlight %}

This results in the following output

```{"A2"=>"42.706849315068503", "B2"=>42.7068493150685}```

If we look at this more closely, the last two digits(03) get trimmed.

Next time I changed `Data 2` to Number with 0 decimal precision.

![](/img/blog/creek_excel/creek_number.png)

I run the following code.

{% highlight ruby %}


gem 'creek', '~>1.0.8'
gem 'awesome_print'

require 'creek'
require 'awesome_print'

creek = Creek::Book.new ("datafile/excel_with_number_formatting.xlsx")
sheet = creek.sheets[0]
ap puts sheet.rows.to_a[1]

{% endhighlight %}

This results in the following output

```{"A2"=>"42.706849315068503", "B2"=>42}```

So, looks like with the value of 0 precision, creek gem convert this to type int.


Ideally there should be an option where we should be able to seperate excel data types and the raw excel with creek.

The complete source code is available [here][creek-example]

{% if true %}
  <div id="disqus_thread"></div>
  <script>
    var disqus_config = function () {

    this.page.url = "http://abhinavdhasmana.in/ruby/2015/09/14/ruby-creek-gem-and-excel-formatting.html"; // Replace PAGE_URL with your page's canonical URL variable
    this.page.identifier = "ruby/2015/09/14/ruby-creek-gem-and-excel-formatting.html";
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



[creek-github]: https://github.com/pythonicrubyist/creek
[creek-example]: https://github.com/abhinavdhasmana/creek_examples

