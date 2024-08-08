---
title: Node JS for backend programmers

categories: 
  - Frameworks
  - Programming
published: true
---


I started working in a new and exciting start up, and the thought of using Java for development crushed my soul, so I began shopping around for a fast-development, fast-deployment, easy ramp up and heavily supported language. I wanted something that will be easy for new developers to learn, and most importantly, something that I’ll have fun writing in.</span>

NodeJS was definitely not my first (or my second, for that matter) option: I’m a backend kind of guy. I’ve done some web, but I would hardly claim expertise in web and UI. When I’ve heard of NodeJS, which was described to me as “Javascript for backend”, I was hardly excited about it. I’ve had some experience with JS, and the experience has hardly made me want to keep using it, let alone write my backend with it - it had none of the order or the elegance of what I used to think of as “Real languages”.

However, since the name NodeJS kept coming up from people and colleagues whose opinions I really appreciate, I’ve decided I can’t keep ignoring it and I have to look into it before I dismiss it.

To make a long story short, I now use NodeJS as part of the production environment, and I find it flexible, easy to deploy and very well supported by the community. However, it doesn’t fit everything and everyone - specially if you have a cpu intensive app.

Bottom line - It’s a very efficient **functional language** with **non-blocking I/O** that runs on a **single thread**, on a single cpu (but you can start several apps and communicate between them) and with a **large and thriving community**.

## So, what is NodeJS, anyway?

Basically, Node JS is frameworks designed around [an event-driven, non-blocking I/O model][1], based on Javascript with I/O libraries written in C++. There are 2 main reasons why JavaScript was chosen as the programming language of NodeJS:

  * Google’s V8 engine (the javascript interpreter in chrome)
  * The fact that it had no I/O modules, so the I/O could be implemented a new to support the goal of non-blocking I/O.

The fact that it’s running on Google’s V8 also ensures that as long as google keeps improving Chrome’s performances, NodeJS platform will keep advancing as well.

## What does it do, and how?

Well, from my POV, the main deal with node is that everything is non-blocking, except your code. What does it mean? It means that every call an I/O resource - disk, db, web resource or what have you is non-blocking. How can it be non blocking, you ask? everything is based on callbacks.

So for example, if you’re working with mongodb, this is how a query looks like:

```javascript
var onDBQueryReturn = function(err, results) { console.log(“Found “ + JSON.stringify(results) + “ users”}
console.log(“Calling query”);
db.usersCollection.find({‘_id’: “1234”}, onDBQueryReturn);
console.log(“Called query”);
```

The output will be:

```javascript
Calling query
Called query
Found {“_id”: “1234”, “name”: “User1”}
```

Now, this might make perfect sense for people who are experienced with functional programming, but it’s kind of a strange behaviour for people accustomed to Java, for example. In node (much like in other functional programming languages), a function is a first level object, just as a string - so it can be easily passed as an argument. What happens here is that when the call to the DB is completed, the callback function is called and executed.

The main thing to remember is that node js runs on a single cpu*. Because all it’s I/O intensive calls are non-blocking, it can gain high efficiency managing everything else on a single cpu, because it never hangs waiting on I/O.

## How is it like to write backend app with Nodejs?

In one word - easy. In several words - easy until you get to parallelism .

Once you get the hang of the weird scoping issues of Javascript, Node can be treated like any other language, with a non-customary-yet-powerful I/O mechanism. Multi threading, however, is not a simple as in other languages - but keep in mind that the concept of non-blocking I/O has made the need for multithreading much less necessary than you usually think.

In addition, Node has a very thriving community - which means that everything you could possible want has already been developed, tried and honed: You can use the npm - Node Package Manager (which is Node’s equivalent of the maven repo, I would say) for easy access to everything: One of the most interesting pages is the Most depended upon modules.

### Multithreading in Node

There is nothing parallel in node. It might look like a major quirk, but if you think about it, given that there’s no I/O blocking, the decision to make it single threaded actually makes everything a lot easier - You don’t really need multithreading for the kind of apps Node was designed for if you don’t hang on I/O, and it relieves you from the need to think about parallel access to variables and other object.

Inspite all the above, I feel strange when I can’t run things in parallel, and sometimes it easy kind of limiting, which is why Node comes with cluster capabilities - i.e, running several node processes and communicating between them on sockets - but it’s still experimental . Other options are using the fork(), exec() and spawn(), which are all different implementations of ChildProcess.

## Should I use it or not?

The short answer is, as always, it depends.

If you feel at home with functional programming and you’re running an I/O intensive app - which is the classic use case for a website - then by all means, do. The community is very vibrant and up to date, deployment is a spree (specially with hosting services like [Nodejitsu][2] and[ codeship.io][3])

If you’re running a cpu-intensive application, if you don’t trust dynamic typing, if you don’t like functional programming - don’t use it. Nice as it is, I don’t think it offers anything that couldn’t be achieved using Ruby or Python (or even scala, for that matter).

## Few last tips

(in no special order)

### IDE

IDEs are a kind of religion, but I’ve been using WebStorm and I like it a lot. It’s 30-days trial and a $50 for a personal licence (or $99 for a commercial one), and I think they even provide them for free for open-source projects.

### Testing

It’s very easy to make mistakes, especially in dynamic language. Familiarize yourself with Mocha unit testing, and integrate it into you project from day 1.

### Coordinating tasks

Sometimes you want to run several processes one after another, or you might want to run a function after everything is done, and so forth. There are several packages for that exact purpose, but the most widely used and my personal favourite is Async

### More resources

[Understanding the node.js event loop][4]  
[Multiple processes in nodejs and sharing ports][5]  
[Multithreaded Node.JS][6]

&nbsp;

 [1]: http://nodejs.org/
 [2]: https://www.nodejitsu.com/
 [3]: https://www.codeship.io/
 [4]: http://blog.mixu.net/2011/02/01/understanding-the-node-js-event-loop/
 [5]: http://craigbrookes.com/category/development/node-js/
 [6]: http://oguzbastemur.blogspot.co.il/2013/12/multithread-nodejs.html
