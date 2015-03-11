
I recently ran a small exercise to demonstrate the power, limitations, and workarounds of Node. In the process, I gained a deeper understanding of asynchronous operations, multi-threading, and forking. Many thanks to my mentor, Anoakie, for his valuable help in walking along with me for this exploration.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [What Node does really well](#whatnodedoesreallywell)
- [What Node does not so well](#whatnodedoesnotsowell)
- [What Node does pretty well](#whatnodedoesprettywell)
- [Conclusion](#conclusion)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# What Node does really well

Node was built from the ground up with asynchronicity in mind, leveraging the event loop of JavaScript. It can handle a lot of requests quickly by not waiting around for the request when there are certain kinds of work being done for the request, such as database requests.

Imagine you have a database operation that takes 10 seconds to complete, represented by a `setTimeout`:

```language-javascript
router.route('/api/delayed')
  .get(function(req, res) {
    setTimeout(function() {
      res.end('foo');
    }, 10000);
  });

router.route('/api/immediate')
  .get(function(req, res) {
    res.end('bar');
  });
```

For a back end framework that does not support asynchronous execution, this situation is an anti-pattern: the server will hang as it waits for the database operation to complete and then fulfill the request. In Node, it fires off the operation and then returns to be ready to field the next incoming request. Once the operation finishes, it will be handled in an upcoming cycle of the event loop and the request gets fulfilled.

(In case the `&` in the bash command throws you, it's essentially a way to tell bash to do two things in parallel.)

```lang-bash
# In a synchronous world...
curl http://localhost:3000/api/delayed & curl http://localhost:3000/api/immediate
# this returns 'foo' after 10s and 'bar' right after

# In an asynchronous world...
curl http://localhost:3000/api/delayed & curl http://localhost:3000/api/immediate
# this returns 'bar' right away, and 'foo' 10s later
```

As long as we only write non-blocking code, our Node server will perform better than other backend languages, right? Right?...

# What Node does not so well

JavaScript is a single-threaded scripting language. Compared with other languages where you can spin off multiple threads to do concurrent processing, JavaScript performs work in a single thread.

Node, written in JavaScript, exhibits (suffers?) the same behavior. Some people like single-threaded because it keeps life simple, but it no longer leverages the power of ubiquitous multi-core processors.

Say you have a simple operation inside the Node server which operates on data. It could be simulated with a basic for loop:

```language-javascript
router.route('/api/delayed')
  .get(function(req, res) {
    for (var i = 0; i < 1000000000; i++) {}
    res.end('foo');
  });

router.route('/api/immediate')
  .get(function(req, res) {
    res.end('bar');
  });
```

In this example, asynchronocity will not save you. Run this in a server and it'll take ~2s to process. Imagine your receiving hundreds of concurrent requests for that same operation and you have a major problem on your hands.

```lang-bash
curl http://localhost:3000/api/delayed & curl http://localhost:3000/api/immediate
# Returns 'foo' after 2s and 'bar' immediately after
```

Back end languages like PHP try to address the problem by creating multiple threads in which work can be done. While multi-threading opens up a different can of worms, at least it wouldn't hang in this situation. As a bonus, PHP tries to handle much of the multi-threading for you. (This is based off a very limited understanding of PHP, so corrections here are welcome.)

```lang-bash
# Assuming the server is equivalently written in PHP...
curl http://localhost:3000/api/delayed & curl http://localhost:3000/api/immediate
# Returns 'bar' immediately and 'foo' 2s after.
```

Huh. That's a bummer. Node doesn't handle a commonplace situation nearly as well as PHP. It's beginning to look like a Node production server will need a separate "crunch" cluster to run numbers. Perhaps that separation of concerns (request handling vs. number crunching) is a good idea anyway, but right now it seems like an unfortunate limitation. What can be done about it?

# What Node does pretty well

Node provides an easy way to **fork** multiple instances of Node, creating a cluster. There's [an excellent tutorial on the topic](http://cjihrig.com/blog/scaling-node-js-applications/). Here's the key bit of code, slightly modified for simplicity:

```language-javascript
var cluster = require("cluster");
var numForks = 4;

if (cluster.isMaster) {
  for (var i = 0; i < numForks; i++) {
    cluster.fork();
  }

  cluster.on("exit", function(worker, code, signal) {
    cluster.fork();
  });
} else {
  // Server creation code here
}
```

With this surprisingly small amount of code, you can create multiple forks. A fork is completely isolated, unlike threads; its in-memory store is not shared. Using this setup, the number-crunching code that originally caused the Node server to hang is now gracefully managed by sending subsequent requests off to forks that are available for doing work.

```lang-bash
curl http://localhost:3000/api/delayed & curl http://localhost:3000/api/immediate
# Returns 'bar' immediately and 'foo' 2s after.
```

With the [cluster](http://learnboost.github.io/cluster/) npm module, Node is back in the game and able to handle intensive data operations, much like PHP.

It's worth reiterating that these two approaches (forking vs. threading) are fundamentally different. With forking, you cannot use multiple cores to speed up the processing of a particular operation. But, it can be used to handle multiple request concurrently when they are blocking operations. With threading, multiple cores can, in theory, be used to speed up the operations on a single request.

# Conclusion

It's hard to beat Node at asynchronous operations; while other languages like Ruby and Python support it, Node was structured expressly to optimize asynchronocity.

Meanwhile, the `cluster` module helps Node handle multiple requests with blocking operations concurrently through forks.

As is the case in much of software engineering, choosing one language/framework/technology/data structure or another is about a balance of tradeoffs. Node does some things really well, others not so much. Hopefully this post gave some insight into the balance of forces in Node.