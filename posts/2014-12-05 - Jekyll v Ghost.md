I've splintered off a new blog to capture my learnings, musings, and stories about software engineering and its neighboring topics. Here I will discuss why I set up a new blog and why I chose Ghost as the platform.

# Why a technical blog

I hope to write often about what I'm learning at Hack Reactor. Doing so will serve multiple benefits:

* I have a "[rubber duck](http://en.wikipedia.org/wiki/Rubber_duck_debugging)" to see if I can explain a topic succinctly,
* Explaining a topic to someone else solidifies my own understanding,
* It provides an easy reference point if I forget it later,
* I have a platform to show what I'm learning, and
* I can highlight my own projects.

I still intend to update my [personal blog](http://andrewsouthpaw.blogspot.com). I just need a new outlet for everything I will pick up pertaining to programming, so readers of my personal world don't get bogged down my new forays into web development. That said, I will still endeavor to provide sufficient explanations to be generally comprehensible by people with limited development experience. 

# Choosing a platform

Before setting up a new blog, I needed to choose a new platform. Blogger has served me well for many years to host my personal blog, but it would not suffice for this new one. I set out to compare different blogging platforms. I wanted something flexible, so I could include my own HTML/CSS/JavaScript, but lightweight to not get in the way of my writing process. There are many fine articles ([such as this one](http://sixrevisions.com/tools/open-source-blogging-platforms-for-developers/)) floating around on the web comparing blogging platforms for programmers. I will not repeat them here; instead, I will focus upon the two final contenders and how I made my choice: Jekyll + GitHub Pages, or Ghost.

### Jekyll + GitHub Pages

First, an explanation of Jekyll and GitHub Pages.

![Jekyll](http://jekyllrb.com/img/logo-2x.png)

[Jekyll](http://jekyllrb.com) magically transforms things into files that make up a website. Quite often, those "things" are Markdown files. Markdown is a plain text standard that encodes formatting through special flags, e.g. wrapping something with \` translates it into `code`. It places focus on content, gives the writer some control over basic formatting concepts, and leaves the finer point of what that formatting should look like downstream in CSS files. Easily portable and parsed into HTML files, it's a popular format for bloggers. While Jekyll creates the website files, they still need to be put up on the web somehow. 

![GitHub Pages](http://blog.petegoo.com/images/github.pages.jpg)

That's where [GitHub Pages](https://pages.github.com/) steps in, providing an easy way to publish those files for free on the web. GH Pages offers a shockingly easy solution to hosting basic web sites and demo projects. It took me maybe 15 minutes to figure out how it worked and publish a test project. Essentially, all you need is your project to be on GitHub, you create a branch called `gh-pages`, and you're done. Put any files you want in there and it'll magically appear on the web.

I really liked Jekyll + GitHub Pages when I first came upon it, and so do many others: it is consistently recommended as a blogging platform among programmers. The setup leverages git -- a defacto tool among coders -- so there's a certain eloquence in programmers using this arrangement for their blogs. 

One of my [favorite posts](http://karloespiritu.com/choosing-a-new-markdown-blog-platform/) evaluating this setup mentioned the complexity of workflow: three windows (an editor, a compiler, and a preview), plus committing every time you wanted to push a change. This held some sway for me, but not nearly as much as the next factor.

Jekyll + GitHub Pages serves up "static" pages. I could publish a project through this method, so long as I didn't use a server, database, or really perform anything on the backend. Since Hack Reactor is a full-stack school (meaning it teaches both front- and back-end web development), this struck me as quite limiting. Even now, I have a project that uses a node backend, and quite soon I will be building more in school. Those projects could not be hosted by GitHub Pages, so then I would need to devise a way to tie them all together on my website/blog. Pretty soon, I was looking at a rather complicated picture. Then, I stumbled upon Ghost.

### Ghost

![Ghost](http://tryghost.org/ghost.png)

[Ghost](ghost.org) launched from a Kickstarter campaign a little over a year ago and has rapidly gained traction among techies. It provides an elegant and clean interface, photo uploading, and managing multiple users on the same blog. It represents a drive "back to the roots" of blogging -- long-form, thoughtful, content-rich -- and away from the echolalia of Tumblr.

Ghost runs on Node, which we will eventually learn at Hack Reactor. I find it poetic to use a blogging platform based on technologies that I will eventually study. I like the platform interface, the live preview of my Markdown file (though I often compose first drafts in SublimeText's distraction-free mode), the photo uploading, it was all quite well and good.

Ghost offers free and paid plans where they host your blog -- much like ordinary blogging platforms. Or, you can take their source code and install it on your own server for free. This meant any projects requiring a backend (i.e. where I'd need to host them on my server) would now conveniently live alongside my blog. While it's certainly possible to arrange matters so they don't all sit on the same server, I preferred the simple mental picture of everything living under the same roof.

Confused by "host it yourself"? Think of it like this: the platform -- the login page, the themes, the database, etc. -- is a huge package of code, it just needs a server to live on to appear to the Internet. Whether that package of code is on servers run by Ghost, or run by you, it doesn't matter; the experience is exactly the same. Neat, huh? Such a possibility had never crossed my mind before, yet it makes perfect sense. Such is the beauty of open-source projects. 

Hosting Ghost yourself is not always a wise option. You still have to pay for a server, you have to set it up, you are solely responsible when it goes down. So why bother? There's no financial or time-saving incentive for a small-potatoes blogger like me. Why do it the hard way?

# Why I did it the hard way

I like to understand how things work. I want to be a full-stack web developer, so I need to know how to make servers operate. Neatly packaged services such as Heroku can conveniently hide such details from view in return for some money: you just give them your project code and *voilá* you have an app running on the internet. But I don't want to be beholden to such platforms. They do have their limitations and ultimately I'm limited by a lack of knowledge. Once I know how to do something, I'll decide whether it's worth paying someone else to do it in the future.

True to expectation, setting up Ghost on my server prompted many discoveries about the mechanics of servers, IPs and ports, nginx, and NodeJS. More in a subsequent post.