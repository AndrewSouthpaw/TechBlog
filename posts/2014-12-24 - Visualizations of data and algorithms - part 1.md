
Inspired by the capabilities of the [D3](http://d3js.org/) library, I have embarked on a project to [visualize common data structures and algorithms](http://www.andrewsouthpaw.com/Visualizations-of-Data-and-Algorithms/). So far, I have created demonstrations for bubble sort and quicksort. This post will recount an interesting design challenge I encountered.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
---
# Table of Contents  
*generated with hacked version of [DocToc](http://doctoc.herokuapp.com/)*

- [The problem: how to pause JavaScript for animation](#theproblemhowtopausejavascriptforanimation)
- [Attempt #1: "Pausing" JavaScript](#attempt1pausingjavascript)
- [Attempt #2: Nested `setTimeout` functions](#attempt2nestedsettimeoutfunctions)
- [Attempt #3: Custom library](#attempt3customlibrary)
- [Attempt #4: store animations to playback later](#attempt4storeanimationstoplaybacklater)
- [Future possibility: queuing animations](#futurepossibilityqueuinganimations)
- [Future work](#futurework)
- [Final thoughts](#finalthoughts)

---

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# The problem: how to pause JavaScript for animation

JavaScript runs asynchronously by design. On a basic level, once you've executed a line of code, it will move on to the next line, even if you've said in the previous line that you want to wait three seconds before executing its contents. (This is, essentially, what `setTimeout` does.) When I write something like this...

```language-javascript
setTimeout(function(){
  console.log('started');
}, 1000);
console.log('done');
```

... the output will be...

```language-javascript
done
started
```

Neat, weird, brilliant, annoying, there are many ways to view the asynchronous nature of JavaScript. In my case, I found it annoying because it created a sneaky problem for me.

Take a command where I tell D3 to animate a transition: move the object from its current location to point A. The specifics aren't important; what matters is that I'm telling it to complete the animation over the course of one second.

```language-javascript
d3.selectAll('rect')
    .data(data, function(d) {return d;})
    .transition().duration(1000)
    .attr('x', function(d,i) {return i * BAR_WIDTH;});
```

That's all well and good until you issue another command to make another, different transition shortly after the first one. Remember, JavaScript doesn't patiently wait for the first transition to finish. Instead, it chugs along and executes the second command for a different transition. D3 is now trying to do two animations at once. 

Now, take this to an algorithm, with a whole bunch of animations. See if you can see it work on quicksort:

![quicksort without delays](/content/images/2014/12/allAtOnce.gif)

Did you see the animations? Neither did I. 

The asynchronous nature of JavaScript makes animating an algorithm a pain. Give it an array of 100 numbers to sort, it'll crank through it in an imperceptible amount of time. To animate, we must manually "pause" the execution to animate the intermediate steps and allow them to run their course.

# Attempt #1: "Pausing" JavaScript

You can place a hard (and ugly) halt to JavaScript, forcing it to spin its wheels for a specified amount of time. This code should never be used in deployment.

```language-javascript
function sleepFor( sleepDuration ){
    var now = new Date().getTime();
    while(new Date().getTime() < now + sleepDuration){ /* do nothing */ } 
}
```

I'm almost ashamed to say that I tried this solution. Ugly, yes, but maybe it'd get the job done... Nope. In addition to halting JavaScript, it also halted any animation work. I was stuck with the same problem as before.

# Attempt #2: Nested `setTimeout` functions

Commonly offered as a better alternative to the desired "sleep" function, one can nest calls to `setTimeout`.

```language-javascript
function doStuff() {
  //do some things
  setTimeout(continueExecution, 10000) //wait ten seconds before continuing
}

function continueExecution() {
   //finish doing things after the pause
}
```

This gets the job done. However, implementing this solution quickly gets hairy. All the nesting obscures the flow of logic, and the more animations the more nesting is required. It begins to feel like you're doing backbends to make it work and such gymnastics shouldn't be necessary.

Here is my bubble sort implementation using this solution. If you want, you can delve into the logic, but it won't enhance any understanding -- it's here for anyone's curiosity. 

```language-javascript
var bubbleSort = function(data) {
  if (data.length < 2) return data;
  var sorted = true;

  // Recursive approach to call iterations within setTimeout delay
  function iterate (i) {
    if (i >= data.length) {
      return sorted ? data : bubbleSort(data);
    }

    // Show bars being compared
    highlightBars([data[i - 1], data[i]]);

    // Call remainder after animation for highlightBars
    setTimeout(function() {
      // Set default delay
      var delay = 10;

      // Compare elements, swap, and animate
      if (data[i - 1].num > data[i].num) {
        var temp = data[i];
        data[i] = data[i - 1];
        data[i - 1] = temp;
        sorted = false;
        delay = ANIMATION_DURATION; // set delay to wait for swap animation
        update();
      }

      // Call remainder after animation for swap
      setTimeout(function() {
        clearHighlight();

        // Call remainder after animation for removing highlights
        setTimeout(function() {
          iterate(i + 1);
        }, 250);
      }, delay);
    }, 250);
  }

  // Initiate recursion
  iterate(1);
};
```

Key takeaway: a complicated algorithm with more animations will create an unsightly "pyramid of doom." This solution would not work in the long run.

```language-javascript
function validate() {
   log("Wait for it ...");
   // Sequence of four Long-running async activities
   setTimeout(function () {
      log('result first');
      setTimeout(function () {
         log('result second');
         setTimeout(function () {
            log('result third');
            setTimeout(function () {
               log('result fourth')
            }, 1000);
         }, 1000);
      }, 1000);
   }, 1000);
 
};
```
([Source](https://keyholesoftware.com/2014/07/23/javascript-promises-are-cool/))

# Attempt #3: Custom library

JavaScript appeared to be lacking what I needed: a clear way to create nested `setTimeout` calls sequentially in code.

I built a custom library to do exactly that. Nothing fancy, but filled the need. Titled "andThen," the basic principle was to use an object to accumulate the delay timer, allowing you to create `setTimeout`s that would execute in a sequential fashion. You call `andThen.doThis()` and pass in a callback function.

**andThen.js**

```language-javascript
var andThen = {
  delay: 0,
  doThis: function(cb, delay){
    andThen.delay += delay;
    setTimeout(function() {
      cb();
    }, andThen.delay);
  },
  reset: function(){
    this.delay = 0;
  },
}
```

I was pretty excited about this solution. It made bubble sort look cleaner.

```language-javascript
andThen.doThis((function() {
  // highlight bars being compared
  // ...
})(),ANIMATION_DURATION)

andThen.doThis((function() {
  // swap elements if needed, animate swap
  // ...
})(), ANIMATION_DURATION);

andThen.doThis(function() {
  // remove highlight
  // ...
}, ANIMATION_DURATION)
```

I was especially proud of the name.

Emboldened by my dandy solution, I put it to work on quicksort. The result was... disheartening.

```language-javascript
// Set whole range to gray
andThen.doThis(function() {
  highlightBars(svg, data, 'grey');
}, ANIMATION_DURATION);

// Show range in consideration
andThen.doThis(function() {
  highlightBars(svg, data.slice(l,r), 'yellow');
}, ANIMATION_DURATION * 3);

// Select pivot
pivotIndex = choosePivot(l, r);
andThen.doThis(function() {
  highlightBars(svg, [data[pivotIndex]], 'blue');
}, ANIMATION_DURATION);

// Move pivot to start of array
andThen.doThis(function() {
  swap(pivotIndex, l);
  update(svg, data);
}, ANIMATION_DURATION);

// Partition the array
andThen.doThis(function() {   //  ಠ_ಠ  This is the worst part...
  partitionPoint = partition(l, r);
  andThen.doThis(function() {
    // Recursively sort
    andThen.doThis(function() {
      qsort(l, partitionPoint);
    }, ANIMATION_DURATION);
    andThen.doThis(function() {
      qsort(partitionPoint + 1, r);
    }, ANIMATION_DURATION);
  }, ANIMATION_DURATION);
}, ANIMATION_DURATION);
```

Ugh. It was getting repetitive to the point of being inane (and then, do this... and then, do this... and then, BLAGH!), distracting any reader from the flow of the program. I was running into obscure timing bugs that were all the more difficult to suss out due to the muddying `andThen` code.

The whole feeling like this was not the silver bullet after all, so I went to the interwebs for recommendations. Surely this issue had come up before.

# Attempt #4: store animations to playback later

My fourth and final attempt was inspired by the [work of Mike Bostock on quick sort](http://bl.ocks.org/mbostock/6dcc9a177065881b1bc4). He stored snapshots of the array undergoing the sorting process -- one snapshot after every swap. At the end, the code would cycle through the snapshots and recreate the action in D3.

I took this a step further and turned the collection into "steps" instead of merely "swaps." I wanted to be able to record highlighting of specific bars to show the progression of quick sort.

My solution was to store "command objects" with a handful of basic commands: `swap`, `highlight`, and `clear`. As the algorithm progressed, it would push such objects into the collection. 

```language-javascript
function parseStep (svg, step) {
  var cmds = {
    'swap': function() {
      // update SVG using step data
    },
    'highlight': function() {
      // highlight bars specified on step
    },
    'clear': function() {
      // clear highlight
    }
  }
  if (typeof cmds[step.cmd] !== 'function') {
    throw 'parseStep: invalid command';
  }
  return cmds[step.cmd]();
}
```

After the algorithm finished, I "played back" the steps. The "play back" function uses recursion: play the step, then recurse after a specific duration on the rest of the steps. Doing so allowed the duration to be adjusted dynamically by the user; the adjustments would reflect immediately in the timing of animations.

```language-javascript
function animateSteps (steps) {
  if (steps.length === 0) return;
  parseStep(steps[0]);
  setTimeout(function() {
    animateSteps(steps.slice(1));
  }, ANIMATION_DURATION);
}
```

A recursive approach gave me much better control over the playback experience. `setTimeout`s are created one by one, rather than all at once in an iterative approach. 

![animation changing speed](/content/images/2014/12/animationChangingSpeed.gif)

I hope to add features to it such as rewinding and pausing the animation. By storing all the animation actions, this will actually be possible.

As an added benefit, the code reads much more cleanly. Here's one section of the quicksort algorithm, in which the pivot is selected and moved to the front, the array is partitioned, and then the left and right portions are recursively sorted. Note how the `steps.push(/* yadda yadda */)` does not obscure the flow nearly as severely. 

```language-javascript
function qsort (l, r) {
  if (l < r) {
    steps.push({cmd:'clear'});
    steps.push({cmd:'highlight', color:'green', data:data.slice(l, r)})
    var pivotIndex = choosePivot(l, r);
    steps.push({cmd:'highlight', color:'firebrick', data:[data[pivotIndex]]});
    swap(data, pivotIndex, l);
    steps.push(data.slice());
    var partitionPoint = partition(l, r);
    qsort(l, partitionPoint);
    qsort(partitionPoint + 1, r);
  }
}
```

We are no longer distracted by the delicate timing of all animations -- that is handled later by another function. All we concern ourselves with is noting animations and their order. Where appropriate, the animations could even be refactored out from the algorithm.

# Future possibility: queuing animations

One final idea -- not yet explored -- would be to implement a queue for animations. My current implementation is, in a sense, a queue that begins processing once the algorith finishes. However, it would be possible to set the queue to dynamically begin processing animation commands as they come in. It would not offer much benefit in pausing and rewinding the animations, but it could be a fun mental exercise.

# Future work

* Implement pause and rewind functionality on animation
* Refactor code to separate out D3 handlers, turning it into code easily leveraged by other visualizations. (Currently everything exists in [one file](https://github.com/AndrewSouthpaw/Visualizations-of-Data-and-Algorithms/blob/master/algorithms/quicksort/quicksortVisualizer.js).)
* Eventually make things prettier, but right now I'm aiming for the MVP.
* More visualizations! Move onto data structures for a bit and learn about the [D3 layouts](https://github.com/mbostock/d3/wiki/Layouts).

# Final thoughts

My journey in visualizing data structures and algorithms has only begun. I've gained valuable experience in the process and practiced making critical design decisions, namely how to slow down the visualization of something that executes in milliseconds.

My hope is to provide meaningful, pretty ways to visualize the shape and behavior of data structures and algorithms. I'll never remember the finer points of quick sort, but I'll certainly remember the visual of the array being separated into two sections -- one less than and one greater than the pivot -- moving the pivot in between these two sections and recursing. With that understanding, I should always be able to recreate the algorithm. My goal is to provide similar understanding to other current and future practicioners of code.
