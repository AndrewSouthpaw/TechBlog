![](http://brunch.io/images/others/gulp.png)

Gulp. Without understanding how it works, reading the file can make programmers perform its namesake. (Still, it's better than the equivalent for Grunt. Oh snap!)

Gulp automates tasks. From minifying JavaScript to injecting vendor dependencies into your template `index.html` file, Gulp offers a way to do just about anything. Here we'll cover some of the basic principles of gulp to get you started on your own adventures.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Gulp operates with streams](#gulpoperateswithstreams)
- [Creating a gulp task](#creatingagulptask)
- [Processing the stream](#processingthestream)
- [Organizing your gulp file (beginner version)](#organizingyourgulpfilebeginnerversion)
- [More to come](#moretocome)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Gulp operates with streams

[Joel Cox](http://joelcox.io/) offers a wonderful analogy for streams:

> Streams are like a garden hose. You hook one end to a source, and one end to a destination. Like siphoning gas from a car, you only need to provide enough suction to get the flow started; it'll keep moving on its own accord after that.

In gulp, our source will (generally) be files we want to work with, and the destination will (generally) be where we want those files to go. Connecting these two points is a `pipe`, or possibly multiple pipes.

(Note: for these code snippets, I am assuming you have already installed `gulp` and required it in your `gulpfile.js`.)

Here's a really simple illustration of a stream that will move `.js` files from the directory `./js/` to `./build/js/`:

```language-javascript
var src = gulp.src('./js/**/*.js');
var dest = gulp.dest('./build/js/');

return source
  .pipe(dest);
```

The sources for streams can be varied. Perhaps, instead of the *contents* of the files, you want the *pathnames* of the files. In that case, you could write:

```language-javascript
var src = gulp.src('./js/**/*.js', { read: false });
```

# Creating a gulp task

You create a gulp task, which can then be run from the command line with `gulp [taskName]`, with this wrapping:

```language-javascript
gulp.task('moveJs', function() {
  var src = gulp.src('./js/**/*.js');
  var dest = gulp.dest('./build/js/');

  return source
    .pipe(dest);
});
```

# Processing the stream

Gulp can do much more than just pipe files from one location to another. The `pipe` can be used to pass these streams into certain functions that process the stream in some way. Returning to the hose analogy, these functions are like filters that you place at points in the hose to change the nature of the gas being suctioned from the car.

Below is an example of using `gulp-concat` to join together multiple files and rename it to `app.all.js` while performing the above action.

```language-javascript
var concat = require('gulp-concat');

var src = gulp.src('./js/**/*.js');
var dest = gulp.dest('./build/js/');

return source
  .pipe(concat('app.all.js'))
  .pipe(dest);
```

As you can see, the module `gulp-concat` takes a stream and returns a stream, but changes the nature of the stream in the process.

Naturally, you can have multiple "filters." Here we'll also minify the JS code after joining the files.

```language-javascript
var concat = require('gulp-concat');
var uglify = require('gulp-uglify');

var src = gulp.src('./js/**/*.js');
var dest = gulp.dest('./build/js/');

return source
  .pipe(concat('app.all.js'))
  .pipe(uglify())
  .pipe(dest);
```

On some occasions, the stream is besides the point for the gulp plugin. Take, for instance, `gulp-shell`, which doesn't necessarily need a meaningful stream; its power lies in the commands it will execute while "processing" the stream. In the snippet below, we perform some repeatable steps before team members push their feature branches to the shared repo. (If you're thrown by `nconf`, check out my [post on environment variables](http://www.andrewsouthpaw.com/2015/02/08/environment-variables/).)

```language-javascript
var shell = require('gulp-shell');

nconf.argv();
if (!nconf.get('b')) {
  console.log('Please specify a branch name: --b name');
  return;
}

return gulp.src('')
  .pipe($.shell([
    'git checkout develop',
    'git pull origin develop',
    'git checkout ' + nconf.get('b'),
    'git rebase develop',
    'git push origin ' + nconf.get('b')
  ]))
```

# Organizing your gulp file (beginner version)

Before you know it, your gulp file will be an mess of 500 lines of code. It'll still be easier to read than a Gruntfile, but you're left scrolling through countless lines to discover whether a certain functionality exists.

I plan to write a more detailed post on organizing your gulp tasks, but here's a basic way to get started: hide implementation details with named functions. Borrowing a leaf from John Papa's AngularJS style guide:

```language-javascript
// Moves JS files into build directory
gulp.task('moveJs', moveJs);

// Preps feature branch for pull request
gulp.task('git:pr', gitPr);

/////////

function moveJs() {
  ...
}

function gitPr() {
  ...
}
```

Organizing your code in this fashion will allow any user to quickly understand what functionality already exists in your gulp setup. If they need to know more, they can search for the function in the code.

# More to come

Stay tuned for more posts on gulp. I find the tool powerful and flexible, one that improves the productivity of engineers both solo and in teams. I plan to create more posts on advanced topics of gulp.