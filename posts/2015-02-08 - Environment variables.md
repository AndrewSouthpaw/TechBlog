
Environment variables are essential in practically any web app. If you ever use an API where they give you unique credentials, you need them. If you want automation of any kind, you need them. If you want to be able to deploy your app locally and on Heroku, you... well, you get the idea.

Configuring environment variables is something everyone expects you to know, but few actually teach. My hope with this post is to bridge that gap.

Generators like Yeoman and Slush often do the work for you, magically creating a setup for you to configure your environment variables. However, knowing how to set up a system from scratch will empower you to customize or break free from generators as necessary. I'll cover what I learned through developing my own system for [a project at Hack Reactor](https://github.com/MildHawk/Archivr).

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [What are Environment Variables](#whatareenvironmentvariables)
- [Setting Environment Variables](#settingenvironmentvariables)
  - [From a file](#fromafile)
  - [From the process](#fromtheprocess)
  - [From arguments](#fromarguments)
  - [Summary](#summary)
- [Configuring environment variables](#configuringenvironmentvariables)
  - [The `index.js`](#theindexjs)
  - [The environment files](#theenvironmentfiles)
- [Sharing your local environment variables](#sharingyourlocalenvironmentvariables)
- [Accessing your configuration variables](#accessingyourconfigurationvariables)
- [Conclusion](#conclusion)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# What are Environment Variables

The premise of **environment variables** is straightforward: they are variables that may vary based on the environment. **Environment** here means deployment environment; canonical environments include development and production. 

Settings are likely to change from one environment to another, hence the need for variables. The URI to your database, for example, will likely differ: in development (on your local machine) it could be `mongodb://localhost/db`, while in deployment it could point to your Heroku MongoLabs add-on, e.g. `mongodb://heroku_app...`.

In your `db.js` file where you create a connection to the database, you probably started off writing something like this:

```language-javascript
mongoose.connect('mongodb://localhost/db');
```

That works fine for local, but for it to work on Heroku you would need to manually change it before deployment. That gets annoying quickly. We need a way to have such information be updated dynamically based on the environment. Hence, environment variables.

While the premise is easy, the implementation is where it gets hairy. Buckle up, because we're in for some chop.

Environment variables can be set from different sources. We'll cover different ways to set these variables, then dive into how to use these environment variables to configure your deployment (e.g. local machine vs. Heroku).

# Setting Environment Variables

## From a file

On a local machine, you're likely to define these variables in `.js` files and `.json` files.

In JavaScript, you could define it directly like any normal variable:

```language-javascript
var domain = 'http://localhost:3000';
```

Similarly for JSON:

```
{
  "DOMAIN": "http://localhost:3000",
}
```

Notice the difference in capitalization? This is done to distinguish the source from the variable that will be available inside your code. With a JSON file, you need to still extract the data and set it to variables.

I discovered a nifty npm module called **nconf** provides this functionality:

```language-javascript
var nconf = require('nconf');

// Load environment variables from file
nconf.file({ file: __dirname + '/local.env.json' });

// Access them and set to variables in your code
var domain = nconf.get('DOMAIN');
```

Best of all, `nconf` doesn't throw an error if the file doesn't exist. (That's important for deploying on production, where you wouldn't have a `local.env.json` file.)

Why all the wrapping, putting them in a file first -- why not just set them directly as variables? It's a valid question, one which will be answered shortly.

## From the process

`nconf` also allows you to access variables in the process's environment, exposed through Node's `process.env` method. When you run something like this:

```
NODE_ENV=development node app.js
```

you are passing an environment variable called `NODE_ENV`, set to `'development'`, to `node app.js`. Then, inside `app.js`...

```language-javascript
console.log(process.env.NODE_ENV); // 'development'
```systems, such as Heroku, DigitalOcean, or Azure, allow you to tell it what environment variables should be set on the process. Using Heroku as an example:

```
heroku config:add NODE_ENV=production
heroku config:add EXPRESS_SESSION_SECRET=mysecret
# and so on
```

Here's how you access these types of variables using `nconf`:

```language-javascript
// Running NODE_ENV=development node app.js

nconf.env();
console.log(nconf.get('NODE_ENV')); // 'development'
```

## From arguments

As a final bonus, it can capture arguments. For example:

```language-javascript
// Running node app.js --env production

nconf.argv();
console.log(nconf.get('env')); // 'production'
```

Capturing arguments by flags is handy (you could use `nconf` to pass flags to `gulp` tasks, for instance), but it's not important in the setting of environment variables.

Another side note: we're only scratching the surface of `nconf`'s functionality. It provides hierarchical access, overrides, defaults, and a bunch of other features that can be useful in larger-scale operations.

## Summary

`nconf` provides convenient, centralized access to environment variables from a variety of sources: arguments, the process environment, and files. It's a gathering place. We'll use the last two sources for our deployment config.

Grabbing environment variables from a JSON file when working locally is convenient and intuitive. It's a lot easier than writing each one out on the command line execution (`NODE_ENV=production DOMAIN=... node app.js`) or putting them in a `.sh` file and using `source`. Those options work as well, I just don't like them as much.

Now that we have a way to grab environment variables, let's focus on using these variables to configure our environment. (Remember, having environment variables is only half of the job: we also need to set them to variables inside the app and make them available to other parts of the system.)

# Configuring environment variables

Credit where it's due: the following strategy I borrowed from the Yeoman full-stack-angular generator.

High-level view: we create a `environment` module, made available to the rest of the server, which will be responsible for setting variable defaults. It will then delegate to files named by their environment (`production.js`, `development.js`, etc.) for custom configuration. We'll then access the module's settings with `var config = require('./path/to/config/environmnt');`.

The file structure we'll set up looks like this:

```
server
|____config
  |____environment
  | |____development.js
  | |____index.js
  | |____production.js
  | |____test.js
  |____local.env.json <-- you've already made this
```

## The `index.js`

As usual, we open with a few modules we'll be using:

```language-javascript
var path = require('path');
var _ = require('lodash');
var nconf = require('nconf');
```

`index.js` begins by loading `nconf`.

```language-javascript
/**
 * nconf sets environment variables intelligently. Will try loading the
 * local.env file if it exists. Will also pull from the existing `process.env`
 * variable. This allows us to use local environment variables provided through
 * local.env, and the Heroku env variables provided through process.env.
 *
 * Format to use nconf is: nconf.get('variableName');
 *
 * Initialize nconf here.
 */
nconf
  // grab flags, e.g. --foo bar --> nconf.get('foo') === 'bar'
  .argv()
  // grab process.env
  .env()
   // load local.env if exists
  .file({ file: __dirname + '/../local.env.json' });
```

We then create an object on which we set defaults:

```language-javascript
var all = {
  env: nconf.get('NODE_ENV') || 'development',
  port: nconf.get('PORT') || 3000,
  // and so on
}
```

We'll export this defaults object extended with another object, exported by the file for our current environment.

```language-javascript
module.exports = _.merge(
  all,
  require('./' + all.env + '.js') || {});
```

Visit [here](https://github.com/MildHawk/Archivr/blob/develop/server/config/environment/index.js) for a complete example of an `index.js`.

## The environment files

Environment files are very similar, in that they export an object with properties we want to access from elsewhere in the app. Here's an example of `development.js`.

```language-javascript
// development.js
module.exports = {
  mongo: {
    uri: 'mongodb://localhost/myAppDb'
  },
  seedDB: true
};
```

As a momentary aside, seeding your database can be a useful and time-saving technique for local testing. You create a `seed.js` file with a bunch of database commands to build out your database. For example, using Mongoose:

```language-javascript
var mongoose = require('../db/index.js');
var User = require('../api/user/userModel');

// Clear database
mongoose.db.dropDatabase();

User.find({}).remove(function() {
  User.create({ 
    username: 'Andrew',
    password: 'password'
  }, function() {
    console.log('Finished seeding database.');
  });
});
```

You can then inform your server to seed the database when `seedDB` is set to `true` in your server file:

```language-javascript
// in app.js...
if (config.seedDB) {
  console.log('Seeding database.');
  require('./path/to/seed');
}
```

Moving along to `production.js`, you'll likely reference environment variables set on your deployment configuration. For example, this configuration would capture a few different common providers of MongoDB, and I'd set one of those on my deployment.

```language-javascript
mongo: {
  uri: process.env.MONGOLAB_URI ||
       process.env.MONGOHQ_URL ||
       process.env.OPENSHIFT_MONGODB_DB_URL+process.env.OPENSHIFT_APP_NAME ||
       'mongodb://localhost/myAppDb'
}
```

Now that you're finished setting up your configuration variables, it's time to use them in your app.

# Sharing your local environment variables

Your deployment environment variables (the ones on Azure, Heroku, etc.) will always be there. The local ones, because they live in a file, need to be shared among your team. **Be sure to `.gitignore` your `local.env.json` file.** Failure to do so will make your secret API keys public. Bad times. 

I typically upload the `local.env.json` to a team Dropbox folder, and remind all team members to download a fresh copy when something changes. Or, you could go real fancy, and explore how to automatically download from a server using Gulp and Node every time you build your app.

# Accessing your configuration variables

Fortunately, this portion is not nearly as difficult, thanks to all the work you did in the previous section. Simply import the module and reference variables off it:

```language-javascript
var config = require('./path/to/config/environment');
console.log(config.mongo.uri); // result depends on your environment!
```

# Conclusion

That covers just about everything you'll need to configure your app for deployment in multiple environments. It will take a long time on your first round, possibly hours to work out all the little glitches and details. But trust me, it gets *way* easier; my second attempt took ~10% of the time. Automating your app configuration gives you flexibility in your testing, saves you time and energy, and shows an appreciation for automation and process development -- a must for any software engineer.

