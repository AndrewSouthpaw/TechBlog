
Writing tests in Mocha/Chai can be intimidating to a junior developer. Here are a few steps I used to make that world less scary. For this post, we will focus on the bare minimum to get you testing quickly using the browser.

# Ground assumptions.

This post assumes you already have Node and npm installed. If you don't, follow one of the many tutorials out to get them.

# Build your template testing folder.

We're going to create a template folder that will contain all the necessary pieces to write tests in the browser. This folder can then be copied into a project or module and it will be self-contained.

### Download the files.

[Mocha](http://mochajs.org/) provides a testing framework for node and the browser. It's an insanely powerful tool. We're going to use a *small* portion of its functionality to write tests that will be run in the browser.

Download these two files: [mocha.js](https://raw.githubusercontent.com/mochajs/mocha/master/mocha.js), [mocha.css](https://raw.githubusercontent.com/mochajs/mocha/master/mocha.css). 

[Chai](http://chaijs.com) provides a handful of shortcuts to *assert*, or test, particular questions. You assert that the thing you're testing conforms to some expected behavior. For example, if you made a function `add` which added two numbers together, you could assert that `add(3,2)` should equal `5`, thus providing a simple test for whether your function works as expected.

Download this file: [chai.js](https://raw.githubusercontent.com/chaijs/chai/master/chai.js).

### Create your boilerplate.

You want boilerplate code to get started quick and easy for each project.

```language-markup
<!DOCTYPE HTML>
<html>
  <head>
    <title>Toy Problems Testing Suite</title>

    <!-- testing frameworks -->
    <link rel="stylesheet" href="lib/mocha.css">
    <script src="lib/mocha.js"></script>
    <script src="lib/chai.js"></script>
    <script>
      mocha.setup('bdd');
      window.expect = chai.expect;
      $(function() {
        window.mochaPhantomJS ? mochaPhantomJS.run() : mocha.run();
      });
    </script>

    <!-- source files -->

    <!-- tests -->

  </head>
  <body>
    <div id="mocha"></div>
  </body>
</html>

```

Save this boilerplate as `SpecRunner.html` along with the other files.

### Organize your files for easy import to a new project.

Here's the structure of my files, inside of a folder called `test`. 

    test
    |___lib
    | |___chai.js
    | |___mocha.css
    | |___mocha.js
    |___SpecRunner.html
    |___tests

With this setup, the `test` folder can be copied into any project or project component.

# Set up your text editing environment (in Sublime).

The syntax of Mocha and Chai still feels wordy, even though they're abstracting a lot of complexity. 

```language-javascript
describe('module #1', function () {
  it('should do this thing', function () {
    expect(...).to.eql(...);
    expect(...).to.equal(...);
  });
  it('should do this other thing', function () {
    expect(...).to.be.a.(...);
  });
});

describe('module #2', function () {
  /* and so on */
});
```

There's a lot of boilerplate `describe`s and `it`s. Let's speed up the process with snippets. Install the "Mocha Snippets" package in Sublime using the package manager. Now you have a bunch of useful snippets, of which I use these the most:

    desc<tab> - describe
    befr<tab> - beforeEach
    aftr<tab> - afterEach
    it<tab> - it

A quick aside about these snippets. By default, they will often provide a `done` argument to the callback:

```language-javascript
it('should do what...', function (done) {
  
});
```

This `done` argument is used for asynchronous testing. That's intermediate-level stuff; for now, we're going to assume you're doing everything synchronously. **NB: Delete that `done` argument from the callback.** If you don't, your tests will fail after a timeout of 2 seconds and you won't understand why. It's confusing and took me a while to figure out; don't fall prey to the same mistake.

Moving along...

I also created these two custom snippets. (Confused about custom snippets? Read this post.)

**expect-to-equal.sublime-snippet**

```language-markup
<snippet>
  <content><![CDATA[
expect(${1:true}).to.equal(${2:true});
]]></content>
  <tabTrigger>exeq</tabTrigger>
  <scope>source.js</scope>
</snippet>
```

**expect-to-eql.sublime-snippet**

```language-markup
<snippet>
  <content><![CDATA[
expect(${1:true}).to.eql(${2:true});
]]></content>
  <tabTrigger>exeql</tabTrigger>
  <scope>source.js</scope>
</snippet>
```

**expect-to-be.sublime-snippet**

```language-markup
<snippet>
  <content><![CDATA[
expect(${1:true}).to.be.a(${2:'boolean'});
]]></content>
  <tabTrigger>exbe</tabTrigger>
  <scope>source.js</scope>
</snippet>
```

# Start testing!

Write out your test files and save them into the `tests` folder. Make sure you include the test files, as well as any necessary scripts, on your `SpecRunner.html`.

Here's a quick rundown of writing and organizing tests. Comments are inline.

```language-javascript
describe('group of tests #1', function () {
  it('test description #2 (e.g. "should add numbers together")', function () {
    expect(thing1).to.equal(thing2); // does thing1 === thing2?
    expect(...).to.eql(...); // does thing1 deepEqual thing2? (good for arrays, objects)
  });
  it('should do this other thing', function () {
    expect(thing).to.be.a.('function'); // is typeof thing a <blank>?
  });
});
```

Using this format, here's an example of the tests I wrote for my bubble sort algorithm.

```language-javascript
describe('bubbleSort', function () {
  it('should have a function `bubbleSort`', function () {
    expect(bubbleSort).to.be.a('function');
  });
  it('should return arrays of length < 2', function () {
    expect(bubbleSort([1])).to.eql([1]);
  });

  it('should sort an array of length > 2', function () {
    expect(bubbleSort([4,5,1,2,3])).to.eql([1,2,3,4,5]);
  });

  it('should handle an array in reverse order', function() {
    expect(bubbleSort([5,4,3,2,1])).to.eql([1,2,3,4,5]);
  });
});
```


Once comfortable with the workflow, it should be possible to get up and running in under a minute. I use tests in almost every situation -- even on toy problems when we're under a time crunch. Writing out a single tests saves you the repetitive task of validating the result after every update of your script, thus allowing you focus on debugging as necessary. (Also, there's something *so* satisfying when you write out a bunch of tests, compose your script, and they all pass the first time around.)

Happy testing!
