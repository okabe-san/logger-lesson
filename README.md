# Logging to improve development
A brief instruction on using loggers to improve development. Uses a MEAN app as a starting point to teach basic logging concepts using [Winston](https://www.npmjs.com/package/winston).

## Why log?
Over and over you're probably heard "don't `console.log()`!" That is good advice, because adding console.logs is a great way to clutter up your code with actions that you won't be able to access in most cases. On the front end, it can work ok, but as soon as you deploy a server, you can't see those logs anymore. Even on the front end the logs are temporary, lasting only as long as someone can look at the console.

But logs are a critical part of debugging, *especially* when something is deployed and you no longer have easy access to console.logs. We want logs that are informative, easily sortable so we can find exactly the logs we want, and stored somewhere we can retrieve them. If you're going to be logging anyways, why not add a good logging system into your development process from the start? DRY!

## Ok, you sold me, how do I start?
### Project installation
First, fork and clone [this basic crud app](https://github.com/dsudia/mean-crud-app). Then

```shell
$ cd mean-crud-app
```

Next, you'll need to have Mongo up and running on your machine. The easiest way to do that on Mac is:

```shell
$ brew install mongodb
```

followed by

```shell
$ brew services start mongo
```

### Poking around
Open up the project in your favorite code editor. Do a find all (CMD+Shift+F in my fav, VS Code) for "console.log"

There are a number of places where console.log is being used; some just put out some basic information about an event like "Connected to database!" Others are outputting an error to the console to help in debugging. These are great use cases for logging.

What other use cases for logging have you run into for logging in your applications?

### Logging concepts
There are several popular logging packages across languages that all follow a general structure, offering five levels of logging. The levels are provided so a user can differentiate how much information they want; they are inclusive, so the most detailed level of logs will include all logs from the four levels above it. The levels are:

1. Error
1. Warn
1. Info
1. Debug
1. Trace

If I read logs only at the Error level, I will only see messages logged at that level. If I search for logs at the Debug level, I will see logs for the Debug, Info, Warn and Error levels.

Have you seen any logs to your console that resemble these levels?

One common place to see logs at the Warn level are in npm installations. If a package is deprecated, that is the level npm logs at. Warn tells you that nothing has gone wrong, but could soon!

#### What level should I choose?
There are no hard and fast rules around what level a particular message should be logged at. There are obvious parallels; an Error from your application should probably be logged at the Error level. But beyond that, it's up to you.

In general, Trace should be left for things that are stepping you through literally every action of your application no matter how inconsequential. I have never seen this outside of large enterprise applications. Debug should be used in places where you are only likely to run into it in the case of a bug, often when interacting with external packages. Info is useful in places where you want to make sure to note an event, but it's only a positive effect, like connecting successfully with a database. Warn is useful in situations where a non-breaking, but non-ideal event occurs, like a user only partially filling out a profile. Error is for errors.

#### Where do my logs go?
In a production environment at a business, you will probably log to a server somewhere, with a UI like [Kibana](https://www.elastic.co/products/kibana) to run analytics on your logs. That requires setting up several tech stacks and wiring them up. We're not going there. For dev purposes, logging to the console (yes, we can still do that!) and to a file for storage are the most common use cases, and, good news, those are built into most popular logging packages.

## Putting it into practice
In the mean-crud-app directory, run

```javascript
npm i --save winston
```

[Winston](https://www.npmjs.com/package/winston) is a popular logging package for Node ([Bunyan](https://www.npmjs.com/package/bunyan) is another). 

Now let's open up `src/server/app.js` in our editor. There are a couple of console.logs in here that we can replace.

At the top of the file where we are setting up our main dependencies, let's add:

```javascript
var logger = require('winston')
```

By default, Winston only logs to the console, so let's do some configuration. Below that, add:

```javascript
logger.add(logger.transports.File, { filename: 'server.log' });
```

Now it will write to a file, so our logs stick around.

Finally, let's head down to where we are logging some stuff to the console in the Mongoose configuration. We have two console.logs here, one for an error, and one for some information. Let's replace these:

```javascript
// nope...
console.log('Error connecting to the database: ' + err)
// should be...
logger.error('Error connecting to the databaase: ' + err)
```

```javascript
// nope...
console.log('Connected to database: ' + config.mongoURI.development)
// should be...
logger.info('Connected to database: ' + config.mongoURI.development)
```

Let's run the app and see what happens.

```shell
$ npm start
```

## Great! What else can I do?
Let's clean up our console. We probably don't need to see every little thing our application is doing in the console. Especially now that we can see details in a file! So let's set our console and file logs to different levels

Go back into app.js and we'll do some more detailed configuration of Winston:

```javascript
var winston = require('winston');
var logger = new (winston.Logger) ({
  transports: [
    new (winston.transports.Console) ({level: 'error'}),
    new (winston.transports.File) ({
      filename: 'server.log',
      level: 'debug'
    })
  ]
});
```

Run the app again. Now we don't see that *.info* log in our console, but it still shows up in the file!

In the terminal, run

```shell
$ brew services stop mongo
```

Run our app again. This time there is an error connecting to the database (because we stopped it) and we get the error in both our console and our file.

If we were working on an extremely complex app and ran into a bug that was hard to pin down, we could start dropping `logger.debug` everywhere, and change our console level to *debug*. The logs in our production environment could only be info and up, but in test they could be at the debug level. We can customize this process now to whatever makes sense for our workflow, rather than being locked into a logger that only logs to one place, at one level.

PS: [Winston-daily-rotate-file](https://www.npmjs.com/package/winston-daily-rotate-file) is a useful plugin for Winston that automatically saves logs to a new file each day instead of endlessly writing to a single file, which is the default behavior. If you implement winston in one of your projects, I highly recommend it!
