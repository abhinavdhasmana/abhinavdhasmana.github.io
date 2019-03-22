---
layout: post
title:  "Writing mocha test cases for async.series with sinon"
tags: Node.js
author: Abhinav Dhasmana
---
Writing test cases can be challenging in node specially when we are new to node. This task becomes even more difficult when we have to write async test cases. If you have one function calling many more functions, its icing on the cake.

Lets take a look at the example on how to tackle such kind of scenario and write unit test case in such scenarios.

Problem Statement: We have a function which essentially calls 5 other functions. We want to test if our main function calls all the relevant
 functions.

This is how the code looks like

```
  save(xlsData, callback) {

    async.series([
      function (cb) {
        this.function1(cb);
      }.bind(this),

      function (cb) {
        this.function2(data2, cb);
      }.bind(this),

      function (cb) {
        this.function3(data3, cb);
      }.bind(this),

      function (cb) {
        this.function4(data4, cb);
      }.bind(this),

      function (cb) {
        this.function5(data5, cb);
      }.bind(this)
    ], function (err) {
      if (err) {
        return callback(err);
      }
      callback(null);
    });
  }
```

Step 1: Create the basic framework for the test

```
'use strict';
const sinon = require('sinon'),
describe('Storage save', function () {
  it('should call all the right methods', function () {

  });
});
```

Step 2: We do not care how each function works/does. So we stub them. `s` is the class to which these functions belong.

```
'use strict';
const sinon = require('sinon'),
describe('Storage save', function () {
  it('should call all the right methods', function () {
  const function1Stub = sinon.stub(s, 'function1');
  const function2Stub = sinon.stub(s, 'function2');
  const function3Stub = sinon.stub(s, 'function3');
  const function4Stub = sinon.stub(s, 'function4');
  const function5Stub = sinon.stub(s, 'function5');
  });
});
```


Step 3: Call the main function `save` and `assert` that the right methods were called.

```
'use strict';
const sinon = require('sinon'),
describe('Storage save', function () {
  it('should call all the right methods', function () {
  const function1Stub = sinon.stub(s, 'function1');
  const function2Stub = sinon.stub(s, 'function2');
  const function3Stub = sinon.stub(s, 'function3');
  const function4Stub = sinon.stub(s, 'function4');
  const function5Stub = sinon.stub(s, 'function5');
  s.save('xlsData');
  sinon.assert.calledOnce(function1Stub);
  sinon.assert.calledOnce(function2Stub);
  sinon.assert.calledOnce(function3Stub);
  sinon.assert.calledOnce(function4Stub);
  sinon.assert.calledOnce(function5Stub);
  });
});
```

This will result in a failed test case. The reason is save method is calling `async` method. This means that we have written sync test for an asyn function. Writing async test case is very easy with mocha. All we have to do is to tell that this is an async test and tell our test case when it is supposed to be complete. Here are the code changes:


Change ```it('should call all the right methods', function () {``` to ```  it('should call all the right methods', function (done) {```

and move all your asserts inside the save callback after `done` has been called. The final code will look like this

```
'use strict';
const sinon = require('sinon'),
describe('Storage save', function () {
  it('should call all the right methods', function (done) {
  const function1Stub = sinon.stub(s, 'function1');
  const function2Stub = sinon.stub(s, 'function2');
  const function3Stub = sinon.stub(s, 'function3');
  const function4Stub = sinon.stub(s, 'function4');
  const function5Stub = sinon.stub(s, 'function5');
  s.save('xlsData', function() {
    sinon.assert.calledOnce(function1Stub);
    sinon.assert.calledOnce(function2Stub);
    sinon.assert.calledOnce(function3Stub);
    sinon.assert.calledOnce(function4Stub);
    sinon.assert.calledOnce(function5Stub);
    done();
  });

  });
});
```

PS: Make sure you restore the stub methods.

{% if true %}
  <div id="disqus_thread"></div>
  <script>
    var disqus_config = function () {

    this.page.url = "http://www.abhinavdhasmana.in/node/2016/02/29/async-series-mocha-test-cases-with-sinon.html"; // Replace PAGE_URL with your page's canonical URL variable
    this.page.identifier = "node/2016/02/29/async-series-mocha-test-cases-with-sinon.html";
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

