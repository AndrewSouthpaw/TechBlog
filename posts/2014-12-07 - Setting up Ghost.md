
After [choosing Ghost as my next blogging platform](http://www.andrewsouthpaw.com/2014/12/05/jekyll-v-ghost-creating-a-new-blog-and-why-i-did-it-the-hard-way/), I embarked on a bold adventure to set it up on my own server.

Fortunately, my server was already up and running. Under the tutelage of the kind and exceedingly talented developer Anoakie, I bought a VPS on DigitalOcean, locked it down with the appropriate firewalls, and learned a lot about Linux and command-line in the process. Up until that point -- which was late September -- I largely regarded the terminal as a place for dark incantations that could turn my computer into a really expensive paperweight. Still, it was a lot to take in; I could probably reproduce all the steps, but I'd be hard pressed to explain what each step accomplished.

I followed a clear and easy tutorial on [manually installing Ghost on your own server](http://www.howtoinstallghost.com/how-to-install-ghost-on-ubuntu-server-12-04/). Unfortunately, programming and servers are complex environments. Tools are required that may be installed slightly differently. Servers may have weird firewalls or setups. There are many different pieces to tie together. If you don't actually understand how they connect, you're likely to run into some mysterious errors because you tried to stick the square jack into the triangle socket.

# My Square Jack in the Triangle Socket

The mysterious bug appeared when switching over to "production" mode. Production mode essentially configures the Ghost platform to prioritize speed and assumes all your details are squared away. While the development environment worked fine, switching to production would no longer load the blog. I received a simple white page along with a cryptic message: "502 Bad Gateway", brought to me by something called nginx.

![My new shiny blog, freshly launched into production... wait, what?](/content/images/2014/12/Screen-Shot-2014-12-04-at-4-00-37-PM-1.png)

What was going on? Clearly nginx had something to do with it.

# Troubleshooting in the Dark

Troubleshooting this bug felt akin to exploring an empty rabbit warren -- dark, cramped, confusing, paths branching in multiple directions, and sadly devoid of fuzzy bunnies. And in this case, given my (lack of) knowledge about servers, I didn't really know what a bunny was, how it lived, or its social habits.

StackOverflow is an amazingly useful forum for any programmer. But, it's like walking into a hospital and saying "My hip sometimes hurts. What's wrong?" Except I'm not asking the question: I'm digging through records of the symptoms other walk-ins have complained about. And I know my hip hurts as well, but there can be many reasons for that problem.

In pursuit of a particular solution, I would invariably open more search tabs to learn about another concept necessary to implement the solution. Often my search would take me to solutions offered by other sysadmins, but the technical level was way above my head. Plus, it was difficult to truly know if my problem was the same; servers are complex environments, as I mentioned. Generally, once I learned the minimum necessary to attempt the solution, it wouldn't fix the problem. StackOverflow can be both helpful and horrible for newcomers to programming.

After much squinting at a screen (as if that would help me understand better) and trying to hold a lot of fresh knowledge in my head at the same time, I was ready to throw in the towel. Just then, Anoakie, my trusty programming mentor, swooped in to save the day! Within minutes, he sussed out the problem.

# What does "Bad Gateway" mean anyway?

Understanding the problem requires some knowledge about how the Internet works. More specifically, how the Internet, servers, websites, and visitors all interact.

I have many thanks to give to Anoakie for his help with developing the following analogy.

### Internet-ville, IP Addresses, and Ports

![Internet-ville](http://www.clker.com/cliparts/D/e/c/K/e/d/city-skyline-hi.png)

The sun rises on Internet-ville, a burgeoning metropolis with a Byzantine collection of roads and overpasses and freeways. Each street in Internet-ville represents a computer/server, designated by an IP address (such as `123.123.123.123`). Because humans are bad with numbers, we often use "domain names" to map to IP addresses, such as `andrewsmithpaw.com`. Each street is dotted with many houses: numbered 1 - 65535, in fact. Many house numbers [are reserved](http://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers#Well-known_ports) officially or by convention; for example, port 80 is where `HTTP` traffic goes. When you fire up your browser and visit `www.google.com` street, you knock on house 80 because of the `http://` in front of it.

Why the multitude of houses? It allows a single street [to serve different applications or proccesses](http://en.wikipedia.org/wiki/Port_%28computer_networking%29) depending on where the user (or the server!) wants to visit. Each house represents a way to send/receive data over the network, often by way of applications. Ghost, for example, by default sets up shop in house 2368 on any street. Number 2368 is not significant, it just hadn't been unofficially reserved by anyone. 

"Reserving?" Two applications don't like to share the same house. It's possible, but it requires a bit of work involving forks and children (how gruesome!). Without special setup, one process will set up their house first, then the housing association denies the second's application to build with a `EADDRINUSE` error. Ghost chose 2368 because it seemed unlikely that anyone else would have already set up a house there. By contrast, if they built it at 5432, they would likely get into land disputes with the PostgreSQL database system.

So, following the Ghost tutorial, I set up `2368 andrewsmithpaw.com`.

### nginx and localhost

Each street is, by default, an ungated, unsecured community. Anyone can go up to any house and walk right in. That is what firewalls (like [iptables](http://en.wikipedia.org/wiki/Iptables)) are for: you can configure which houses are locked or unlocked. Generally, a "tight" firewall would only allow visitation to port 80, the `HTTP` house. But now we're back to the original problem: how can we differentiate applications?

This is where `localhost` enters the picture. Reserved as `127.0.0.1` [for IPv4](http://www.tech-faq.com/127-0-0-1.html), `localhost` offers a convenient way for you to build an apartment complex at a particular street number instead of a single-family residence. `localhost` is a floor on the apartment complex, giving you a new range of ports available as room numbers. For example, a PostgreSQL database system listening on `127.0.0.1:5432` would live on the floor `127.0.0.1` at room `5432`.

Which brings us to a new problem: who decides which rooms can be visited? Maybe the resident of 666 is engaged in dark arts and does not want to be disturbed. (As an amusing aside, port 666 is officially reserved by Doom, the first online first-person shooter. Coders have a great sense of humor.) We now need a way to direct visitors to the appropriate rooms.

Enter Mr. [Nginx](http://nginx.org/en/), the doorman, into the picture. Mr. Nginx controls access to the building. When a visitor walks in, Mr. Nginx kindly goes to the requested room to see if there's a package for the visitor. 

Here's a theoretical example: there exists `blog.andrewsmithpaw.com` and `www.andrewsmithpaw.com`. The former goes to a blog, the latter to a portfolio website. Both point to my server, `123.123.123.123`, and thus visitors for either arrive at house 80. Mr. Nginx, seeing that BlogVisitor came for `blog.andrewsmithpaw.com`, goes up to the floor `127.0.0.1`, gets a package from room 2368 labeled `blogindex.html`, and passes it along. WebsiteVisitor arrives next, with a `www.` sharpied on their t-shirt front. (Tacky, I know.) Mr. Nginx looks them up and down, says, "Ah, yes, your package is right around here," rummages around in a cubby labeled `/data/projects/`, and gives them a package labeled `index.html`. 

Based on the [tutorial to configure nginx for Ghost](http://www.allaboutghost.com/how-to-proxy-port-80-to-2368-for-ghost-with-nginx), I configured the setup described above: visitors to my blog should be redirected to `127.0.0.1:2368`.

# The Problem

Have you spotted the problem with my setup? Now that I understand how all the pieces work, the mistake was obvious; at the time, it was all voodoo magic.

BlogVisitor walks in to the apartment complex at 80 andrewsmithpaw.com. Mr. Nginx goes up, eventually returns empty-handed. "Sorry," he explains, "I knocked on room 2368 but no one answered." Not one to let a person leave with nothing, he hands them a slip of paper with `502 Bad Gateway` written on it. Meanwhile, Mr. Ghost, waiting with cookies and tea for visitors at 2368 andrewsmithpaw.com, becomes increasingly lonely.

(For those that recall the development environment worked, it's because the tutorial did not have me change Ghost from listening at the default location: `127.0.0.1`. Only the production environment was adjusted. D'oh.)

# The Solution

Tell the Ghost application to set up shop at `127.0.0.1:2368`. Like the original defaults. Simple as that. One line of code. 

# Allowing the dust to settle

Tutorials are great and all, but this situation is a perfect example of where a copy-and-paste philosophy easily gets you into trouble. Had I actually understood the way all the components worked, I would have spotted this mistake immediately. The abundance of packaged solutions in programming can be a tremendous boon, but it can also be a curse.

My Ghost blog is now humming along happily on my server. Setting up a Ghost blog should, in theory, take less than 15 minutes to configure on your server. Because of this mistake, it took 4 hours, but I learned much in the process. I now how a mental model of the Internet, IP addresses, ports, localhost, applications, and nginx. My knowledge is nowhere near complete, but it's much further along.

Setting up my Ghost blog was both an opportunity to share my learning about programming and learn about programming at the same time. It was a fun experiment, valuable, informative. I'm glad I did it. Much like the person that carefully rebuilds the engine of their car, I have a certain affection and connection with my Ghost blog now. I think his name is Boo.
