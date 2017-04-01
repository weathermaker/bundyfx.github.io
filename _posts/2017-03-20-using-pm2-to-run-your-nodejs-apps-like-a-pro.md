---
layout: post
section-type: post
title: Using PM2 to run your Node.js apps
category: NodeJS
tags: [ 'DevOps', 'NodeJS', 'PM2' ]
image: /img/posts/2017-03-20-using-pm2-to-run-your-nodejs-apps-like-a-pro/1.png
---

After a weekend hacking away building a new web application, I decided to pick up the popular Process Manager tool [PM2](https://github.com/Unitech/pm2) and give it a try.
I had heard lots of folks giving their praise to PM2 and I wanted to understand the role it played in running web applications, specifically one running the Express framework.

In this post, I want to go give you an introduction to using PM2 and touch on a few of the aspects that make this such a valuable tool for Node developers.

Contents
=================

* [Installing](#installation)
* [Fork Mode](#fork-mode)
* [Cluster Mode](#cluster-mode)
* [Monitoring](#monitoring)
* [Conclusion](#conclusion)

## Installation

Getting started with PM2 is super simple. First thing's first jump on over the [Github Repository](https://github.com/Unitech/pm2) and take a look.
As per normal with anything Node, we're just going to be installing the PM2 package. We can install it globally by running the following:

`please npm install pm2 -g`

Now that the CLI is globally installed you can use it against any of your applications.

You can always keep pm2 up-to-date by running `pm2 update` which will perform a seamless update.

For this post I'm going to be using my example 'Beer Time' Express Web Application [here](https://github.com/bundyfx/psconf-eu2017-beertime).

## Fork Mode

Let's get our app up and running with PM2.
In this case, my entry point is `index.js`, so we can simply start our App by running `pm2 start index.js`.

![pm2](/img/posts/2017-03-20-using-pm2-to-run-your-nodejs-apps-like-a-pro/2.png)

The first thing you might notice is that the console is free to use again after running PM2. This is very different if you simply used to just running `node index.js` and letting the built-in process manager handle your application.
You can simply hit your app as normal to ensure all is working as intended, in this case, `http://localhost:5000/` will produce the default `/` route as expected.

Now that our app is running, let's look into some features of PM2.

Since we didn't pass our app name to PM2 when we started it, it has the same name as our entry point file. We can get the information of our process by running `pm2 show index`.

![pm2](/img/posts/2017-03-20-using-pm2-to-run-your-nodejs-apps-like-a-pro/3.png)

Here we have a ton of information to make the development of our application as simple and performant as possible.

Firstly, we can access our console output from our application by simply doing a **cat** against our `out log path` like so:
`/Users/USERNAME/.pm2/logs/index-out-0.log`

And of course, we can access our error output also within its own file. This helps separate application debug information and offers a history of information that is persistent on disk.

The mode that we're running our application in by default is called **'fork-mode'** which you can see under the `exec mode` in pm2 show output.

Fork mode is simply basic process spawning. This allows us to change the *exec_interpreter*, so that you can run a PHP or a Python server with pm2. The exec_interpreter is the "command" we used to start the child process.
By default, pm2 will use node so that `pm2 start index.js` will do something like:

```javascript
require('child_process').spawn('node', ['index.js'])
```

This mode is very useful because it enables a lot of possibilities. For example, you could launch multiple servers on pre-established ports which will then be load-balanced by HAProxy or Nginx.


## Cluster Mode

PM2 has another mode called: `Cluster Mode`

Since Node is single-thread, only 1 core of your CPU can execute your Node application. Because of this, we can use `-i` or `--instances` which helps by running 1 Node thread on each core of your CPU.

As an example: `pm2 start index.js -i 4`

The Cluster mode will only work with Node as it's *exec_interpreter* because it will access to the [NodeJS Cluster module](https://nodejs.org/api/cluster.html). This is great for zero-configuration process management because the process will automatically be forked in multiple instances.
The above command will launch 4 instances of *index.js* and let the cluster module handle load balancing. This is perfect for simulating a Production load-balanced environment all from your local machine.

You can also pass in `0` as the `-i` value which will automatically spawn as many workers as you have CPU cores.

![pm2](/img/posts/2017-03-20-using-pm2-to-run-your-nodejs-apps-like-a-pro/4.png)

If any of your workers happens to die, PM2 will restart them immediately so you don't have to worry about that either.

## Monitoring

PM2 offers the ability to monitor your application as your developing. This includes such information as Memory, CPU and the Event Loop delay.

This information can be seen when running the `pm2 monit` command.

![pm2](/img/posts/2017-03-20-using-pm2-to-run-your-nodejs-apps-like-a-pro/5.png)

This is a basic look at your application but helps tremendously when trying to pinpoint event loop delays or memory leaks.

However.. Sometimes you just want more monitoring and insight into your application. For this, there is [Keymetrics](https://keymetrics.io/).

Simply sign in with your Github account to get started with a free account. Keymetrics allows us a deep insight into our application running either in a local development environment all the way to Production. As long as your using PM2, you can pass your metrics into the Keymetrics dashboard.

![pm2](/img/posts/2017-03-20-using-pm2-to-run-your-nodejs-apps-like-a-pro/6.png)

As suggested on the dashboard page, simply run `pm2 install pm2-server-monit` to install the monitoring and forwarding aspect of PM2.

![pm2](/img/posts/2017-03-20-using-pm2-to-run-your-nodejs-apps-like-a-pro/7.png)

Once you have installed the required package, simply link your application by passing in your public and secret key by running: `pm2 link secretkey publickey`

Once you've done that, you should see your application being monitored in the console.

![pm2](/img/posts/2017-03-20-using-pm2-to-run-your-nodejs-apps-like-a-pro/8.png)


## Conclusion

PM2 can do so many great things that have traditionally been done by such task runners as Gulp or Grunt. For example, *nodemon* is something we've all used to simply monitor a file path and restart our express app when changes are made.
PM2 makes this easy by offering a `--watch` parameter in which you can simply pass in a path to monitor for changes. If anything happens there, PM2 will restart the application.

If you're finding yourself building more and more Node applications and are curious about process management and how this will work in your Production environment, be sure to check out [PM2](http://pm2.keymetrics.io/docs/usage/process-management/).

