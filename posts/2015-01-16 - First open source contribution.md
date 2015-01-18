I am now designated as a collaborator on an open source project! It's for [DocToc](https://github.com/thlorenz/doctoc), a npm package with a noticeable userbase: ~900 downloads from npm in the last month, and 200 stars on GitHub. This status upgrade happened because I submitted a feature contribution to the project -- my very first.

What follows is that story, with some interesting asides: finding opportunities to contribute to open source projects, the finer points of Markdown and its conversion to HTML pages, and how to gracefully navigate the logistics of open source contributions.

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
---
# Table of Contents  
*generated with hacked version of [DocToc](http://doctoc.herokuapp.com/)*

- [Finding the right opportunity.](#findingtherightopportunity)
- [Documentation contributions.](#documentationcontributions)
- [Feature contributions: Markdown, HTML, and my contribution.](#featurecontributionsmarkdownhtmlandmycontribution)
- [Navigating contribution logistics.](#navigatingcontributionlogistics)
    - [Current practices.](#currentpractices)
    - [Git rebase](#gitrebase)
          - [What it does.](#whatitdoes)
          - [How it works](#howitworks)
          - [How to let yourself make mistakes.](#howtoletyourselfmakemistakes)
- [Final thoughts.](#finalthoughts)

---

<!-- END doctoc generated TOC please keep comment here to allow auto update -->


---

# Finding the right opportunity.

The open source community is huge, making it a land of opportunity and also a land that is seriously intimidating.

![github contributions](http://media.rachelnabors.com/wp-content/uploads/2012/04/github_web1.png)

My favorite way to discover ways to contribute to open source projects is to use them in your own projects. It's easy to interact with other projects because there are *so many* open source tools out there. The ecosystem has grown massive, dedicated to the support of building greater things faster. If you need to do something, there's probably an npm package that does it for you. "There's an app for that," except now the trope applies to programming. 

While learning about or using their tool, pay attention to "Argh!" moments. Such pains points are inevitable and frequently relate to documentation or feature limitations.

# Documentation contributions.

Despite best intentions, the vast majority of projects are woefully under-documented. They are often cryptic and assume a high level of expertise. These documents are often written by the creators, whom were smart enough to create the library, so it's no surprise they may fall short of writing to a less experienced technical level. Although not fundamentally wrong, such documentation makes projects even more impenetrable to beginners.

Most people forget what it's like to not know anything. Do you remember trying to understand how Redis worked without fundamentally understanding the concept of servers? Or `localhost`? Seriously, look at the readme for [Node redis](https://github.com/mranney/node_redis). They don't even mention spinning up a Redis server. *They're skipping a crucial step.* Again, it's not wrong to assume a certain technical level, but it does mean that more beginners are left in the dark. 

So consider improving the "Hello world" documentation. Walking people up to the point where they can do *something* with it. It is famously said that getting to "Hello World" is the hardest part of computer science. One could easily improve the docs by expanding that section, walking along a total beginner to the point where they can start tinkering.

Key takeaway: there are easy ways to add significant value to documentation, beyond cleaning up grammar. While proof reading is valuable, don't sell yourself short by limiting yourself to that scope. 

Meanwhile, keep on the lookout for missing features. This may be hard for an industry standard like Angular. But, sometimes you'll get lucky with a smaller project. This happened to me last Sunday.

# Feature contributions: Markdown, HTML, and my contribution.

I've felt a growing need for a way to generate a table of contents for my technical posts. I could do this manually, but that seemed woefully inefficient. Before embarking on my own project, I decided to search the npm ecosystem. *This is a great way to discover new projects. Any time you're about to create something new, search around for it on the web.* Eventually (after ~20 minutes of digging), I came upon `doctoc`.

Doctoc offers a solution to insert a table of contents directly into your Markdown files. Written in Node, it reads reads the `.md` files in a directory and prepends a table to the front of the file. When Markdown is converted to HTML, its headers are converted into `<h1>` through `<h6>` tags with unique IDs, so the table can also contain links to these elements. (Side note about HTML: you can navigate to any ID on a page by going putting if after a hash mark in your navbar. `www.example.com/index.html#foo` would go to the element with ID `foo` on the page.)

It took me about 20 minutes to familiarize myself with the package and set up an environment for testing. I quickly ran into a curious problem when posting on Ghost: the links didn't work.

Turns out Ghost follows a different set of rules when it generates IDs for headers. While GitHub Markdown replaces spaces with dashes, Ghost removes then outright, and that's just the beginning. Turns out they differ in many subtle ways. 

Bummer. Turns out doctoc would no be my out-of-the-box solution. This was the crucial moment that led me to an open-source feature contribution. I had a few options available to me:

1. Give up and do it manually,
2. Find a different project that maybe did *everything* I needed,
3. Write my own solution from scratch, or
4. Add on to the existing project.

After a time, I was able to get the lay of the land for this code base. Even with good programming design, this process takes a while. Once I understood roughly what was going on, it was time to start breaking things. 

Some people, myself included, learn by trying things out and seeing what happens. Inserting tracer statements to view the input and output, deleting lines to see their impact, changing variables. Setting up a "sandbox" environment for tinkering is abundantly easy with the Chrome dev console for JavaScript.

Node modules, by contrast, have a layer of separation. You can't simply load it up into an HTML page and start testing. It was a cognitive leap for me to figure out how to test Node global modules, even though it's actually straightforward.  Here's how to do it:

1. Global modules install into your `/usr/local/bin` folder. You can navigate there directly using `cd`.
2. Start hacking.

Easy as that! For some reason I treated npm packages as being installed into some shadowy location on my computer, only accessible through obscure words like symlinks and PATH variables. Instead, these packages are very much like any standard collection of JS files. That's the magic of Node -- you can send off packages to be run like executables on your computer that are based off uncompiled script files. You can edit those files and the changes will be reflected immediately, no other special steps. If you irrevocably break your package, you can always uninstall and reinstall through the magic of npm.

All told, it took me about three hours to build out the new feature -- discovery, exploration, design, and testing.

By the next morning, my contribution was approved by the repo owner. They granted me Collaborator status to merge it myself -- there were conflicts due to a more recent commit on the master. A little while later, the update pushed up to npm and now anyone can build a table of contents with links formatted for Ghost.

And so I have become part of the open source ecosystem. While I've theoretically understood the motivation behind most open source work (chiefly, solving a problem that you have), it feels entirely different to go through the process myself.

# Navigating contribution logistics.

GitHub provides a convenient way to edit files in the web platform. It automatically generates a new branch and seemlessly integrates with submitting a pull request. However, you can only change one file at a time with this approach.

If you're editing more than one file -- which is likely on a feature contribution as it will certainly require tests -- your best bet is to fork the repo. With your own copy, you can make multiple file changes in a single commit and then create a pull request.

For success in your pull requests, one must be observant of current practices and comfortable with `git rebase`.

### Current practices.

Items to note:

* What tense and style are used when writing commit messages?
* Are branches merged with fast-forward or no fast forward? (Most obvious indication: if there's an abundance of "merge commit" messages.)
* What coding style do they use? You will want to make your code look like it was written by the same person.
* What testing style do they use? Same reason.

### Git rebase

The infamy of `git rebase` seems to rival Voldemort's. It is uttered in hushed terms, lest `git rebase` overhear your conversation and swoop in to irrevocably bork your repo. 

![git rebase is coming](http://www.mememaker.net/static/images/memes/3741720.jpg)

To make it less scary, let's understand: a) what it does, b) how it works, and b) how to let ourselves make mistakes.

###### What it does.

`git rebase` takes a branch and moves it to be rooted from the current master branch. Visually, it looks like this:

![image of rebase](http://rypress.com/tutorials/git/media/5-1.png)

(Sidenote: although the arrows are pointing to the left, the diagram still reads left to right. The arrows indicate the commit's parent, the thing that precedes it.)

This situation happens quite often. You create a new branch to build a feature. In the mean time, the master moves forward with other commits. When you try to merge back in, you run into merge conflicts. Rebasing brings your branch to be rooted at the current master, so the merge will now be totally smooth. Open source project owners like smooth merges. A lot. Owners will rarely (if ever) deal with merge conflicts -- some will reject your pull request outright, others are nice enough to notify you and give you a chance to fix the issue. 

###### How it works

The mechanics of rebasing are actually pretty straightforward.

1. It takes all your branches commits and moves them to a temporary area.
2. It changes the branch ref to point to master. It's as if you were on master and just entered `git checkout -b <branch>`.
3. It replays the commits back in order onto the branch.

Part three is where the messiness starts, and why the word "rebase" can paralyze developers with fear. Fortunately, the git command line offers lots of helpful prompts to make it easier to troubleshoot.

There are a crazy number of git tutorials out there. If you don't know how to deal with merge conflicts, search around on Google. There are some truly high-quality ones available. (I particularly enjoyed [GitImmersion](http://gitimmersion.com/).)

###### How to let yourself make mistakes.

The scariest part about rebasing is that it seems so... irrevocable. There's no going back to the way the branch was, so don't mess it up!

While git sadly lacks a `git undo` command, there are [creative ways](http://stackoverflow.com/questions/2129214/backup-a-local-git-repository) to backup your git before taking the leap of faith. The most straightforward one is to leverage GitHub. Make sure the GitHub repo is up to date with your own, then execute the command on your local copy. If it breaks, delete the folder and clone down a fresh copy.

# Final thoughts. 

Open source contributions can be a fun game. There's something exhilarating when you can see your work make an impact (however small) in the community at large. 

Unlike several of my previous posts that have more of a tutorial feel to them, this one was decidedly more story-driven. I hope you still found some value in reading it. As always comments are welcome. 







