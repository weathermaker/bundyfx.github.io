---
layout: post
section-type: post
title: Getting to know Consul
category: Service-Discovery
tags: [ 'Docker', 'Consul' ]
---

Recently I had an itching to learn about [Consul](https://github.com/hashicorp/consul). In this post I want to share with you some of my findings and ramble about how this tool works.

![Output](/img/posts/2016-11-26-getting-to-know-consul/logoconsul.png)

Contents
=================

* [Enter Consul](#enter-consul)
* [Key/Value](#key-value)
* [Installation](#installing-consul-on-an-agent)
* [Creating a Service](#creating-a-service)
* [Consul Commands](#consul-commands)
* [Health Checks](#health-checks)


As you can see from the [Readme](https://github.com/hashicorp/consul/blob/master/README.md), Consul is a tool for service discovery and configuration. You might say, *'Service discovery.. pfft, Why would I need that?'* and unless you're getting your hand into the Docker scene you may be correct.

As we pile more and more Services into Docker the chances of services fighting with each other grows. Maybe you've got fifty services running and you need to manage the listening ports for all of these. Add in the associated databases in which some of these services use and you would need to be some sort of magician to remember this all.

One might say: *'why bother specifying static ports in Docker?'*

Sure, Docker can randomly assign ports to your services at run time. However this raises the question, How are your services going to be able to find the other services they need to connect with in order to function.

The concept of Service Discovery is *not* new however it has become more of an essential key to handling dynamically defined Docker environments.

### Enter Consul

Hashicorp's [Consul](https://www.consul.io/) has become a hugely popular tool since it's release a few years back. In this post I wanted to go through an example of its use case and show you how easy it is to work with.

In this post we're going to be working with [Docker](https://www.docker.com/). You can download Docker for [Windows](https://docs.docker.com/docker-for-windows/) or [Mac](https://docs.docker.com/docker-for-mac/).

To start we're going to be working with a Docker image of Consul that's available via Docker Hub titled: [progrium/consul](https://hub.docker.com/r/progrium/consul/). You can simply pull this down by running:

`docker pull progrium/consul`

Once you have pulled down the image we can fire it up:

`docker run -p 8400:8400 -p 8500:8500 -p 8600:53/udp -h node1 progrium/consul -server -bootstrap -ui-dir /ui`

You should have an output like the following:

![Output](/img/posts/2016-11-26-getting-to-know-consul//1.png)

A couple of things to note about how Consul works are displayed in this output:

* *serf*: This is the internal gossip protocol used within Consul. Consul leverages the membership and failure detection features within serf and builds upon them to add service discovery.
* *raft*: [Raft](https://en.wikipedia.org/wiki/Raft_(computer_science)) is a consensus algorithm that is based on [Paxos](https://en.wikipedia.org/wiki/Paxos_(computer_science). Compared to Paxos, Raft is designed to have fewer states and a simpler, more understandable algorithm.
* *consul*: Well.. This is what we're talking about here.

Since we're not running in a cluster *(which is not normal)* you will see an error stating that the node was unable to join the cluster. This is expected with this simple example.

As you can see from image notes on docker hub: *'We publish 8400 (RPC), 8500 (HTTP), and 8600 (DNS) so you can try all three interfaces. We also give it a hostname of node1. Setting the container hostname is the intended way to name the Consul Agent node.'*

This makes it the perfect image for getting started with Consul and understanding its functionality.

We also added the Consul UI switch when we launched our Consul Container. We can access the Consul Console *(ha)* at any time by hitting it in the browser at `http://localhost:8500/`.

![Output](/img/posts/2016-11-26-getting-to-know-consul//2.png)

In the Web interface we get a overview of the current state of our environment plus a whole lot more that you will want to dig around with.

The best and easiest way to interact with Consul to service information is to use HTTP(s).

![Output](/img/posts/2016-11-26-getting-to-know-consul//3.png)

In this example I am using [PowerShell](https://github.com/PowerShell/PowerShell) and hitting the end point with `curl` then transforming the JSON output into an object for easy viewing.

This concept is simple however it allows other applications within Docker networks to call consul to find information about the nodes that are registered. In turn, allowing nodes to easily find each other without conflict.

### Key-Value

In addition to providing service discovery and integrated health checking, Consul provides an easy to use Key/Value store. This can be used to hold dynamic configuration, assist in service coordination, build leader election, and enable anything else a developer can think to build.

This can be a super handy way for Containers to know where they can retrieve important information from such as content/paths and so on.

![Output](/img/posts/2016-11-26-getting-to-know-consul//4.png)

You will see that if we query our *key/value* store we get back a `404 Not found`. Well, we need to first put some data in there before we can get anything back.

This can be done via the web console or via a `PUT` request.

`curl -X PUT -d 'Awesome Information' http://localhost:8500/v1/kv/web/key1`

You will notice you get back a `true` response indicating that the data has been stored. Let's add in a few more.

`curl -X PUT -d 'I like Beer' http://localhost:8500/v1/kv/web/key2`
`curl -X PUT -d 'Kobe Bryant' http://localhost:8500/v1/kv/web/key3?flags=24`

We added two more items into our key/value store!

Notice with the 'Kobe Bryant' Key that we've specified `?flags=24` to the end of our `PUT` request.
This allows us to associate an integer with with a key/value pair that can be used to group certain data together.

`curl -X PUT -d 'Black Mamba' http://localhost:8500/v1/kv/web/key4?flags=24`

As an example we can associate another value and store it with a flag of 24.

We can simply retrieve data from our key/value store in similar fashion.

`curl http://localhost:8500/v1/kv/web/key1`

or in a recursive method *(using PowerShell for pretty output)*:

`curl http://localhost:8500/v1/kv/?recurse | ConvertFrom-Json`

You will notice that the value returned back from these queries is base64 encoded to allow non-UTF8 characters!

In order to get the raw value back from the query you will need to append `?raw` to your query.

`curl http://localhost:8500/v1/kv/web/key3?raw`

Of course, Keys can be deleted as well just as you would expect *(delete all keys)*: `curl -X DELETE http://localhost:8500/v1/kv/web/sub?recurse`

You can read more about Key/Value in Consul [Here](https://www.consul.io/docs/agent/http/kv.html).

### Installing Consul on an Agent

For this example I want to blow away our single node consul server and create a 3 node cluster. We can do that simply by following the instructions on the progrium/consul guide.

`docker run -d --name node1 -h node1 progrium/consul -server -bootstrap-expect 3`
`JOIN_IP="$(docker inspect -f '{{.NetworkSettings.IPAddress}}' node1)"`
`docker run -d --name node2 -h node2 progrium/consul -server -join $JOIN_IP`
`docker run -d --name node3 -h node3 progrium/consul -server -join $JOIN_IP`

And to add in a Client node *(no -server switch)*:

`docker run -d -p 8400:8400 -p 8500:8500 -p 8600:53/udp --name node4 -h node4 progrium/consul -join $JOIN_IP`

![Output](/img/posts/2016-11-26-getting-to-know-consul//6.png)

You can hit the web console to see the nodes listed there.

Installing Consul is so simple because all it requires is just the binaries to run. You can spin up a new ubuntu container and install it within a few seconds, like so:

```
apt-get update
apt-get install unzip -y
apt-get install wget -y
cd /usr/local/bin
wget https://releases.hashicorp.com/consul/0.7.1/consul_0.7.1_linux_amd64.zip
unzip *.zip
rm *.zip
```

Once the consul binaries have been downloaded to the node you can simply run `consul` to see the help.

![Output](/img/posts/2016-11-26-getting-to-know-consul//5.png)

Now that we've got Consul downloaded we can join this agent to our cluster using `consul join` and passing in one of the IP address' of the cluster nodes: `consul join *ipaddress*`.

### Creating a Service

So we've talked about DNS, Key/Value and Installing Consul. What about Services?

All Services can be queried by running:
`curl localhost:8500/v1/catalog/services`

One of the main goals of service discovery is to provide a catalog of available services. You can think of services as an application or database, or really anything that you are offering for consumption by another service or end user.

To iterate over and show how easy it is to setup Consul let's destroy our cluster by running:
`docker rm -f $(docker ps -aq)`

Now let's clone this great repository by [Scott Memberson](https://github.com/smebberson/docker-alpine) that will help demonstrate Services in Consul.

`git clone https://github.com/smebberson/docker-alpine.git`
`cd docker-alpine/examples/complete/`
`docker-compose up -d && docker-compose scale consul=3 app=3`

Once this has downloaded the required images and spun containers up you should be able to hit `localhost:8500` to see the Consul web UI or localhost:80 to see the requests being served to and responded by different Node.js containers, maintaining sticky sessions.

![Output](/img/posts/2016-11-26-getting-to-know-consul//7.png)

These Services were started and bootstrapped to essentially do a `consul join` and connect with the consul cluster. Once joined they can be queried by other services requiring there needs.

To destroy the composition and remove all the containers you can run `docker-compose down`

You can read more about Service definitions [Here](https://www.consul.io/docs/agent/services.html).

### Consul Commands

Consul comes with a slew of commands that agent's can execute. You can get a full breakdown by just typing in `consul` or more detailed help by calling the exact command `consul members -?`

Another example using `consul exec`

`consul exec echo 'hi from $(hostname)'`

![Output](/img/posts/2016-11-26-getting-to-know-consul//8.png)

This sort of functionality is critical as it allows you to `exec` commands on certain tags or even services.

### Health Checks

You may of noticed the health checks in Consul also. This is a critical part of Consul as it's important to know the health of a Service prior to that Service taking on traffic from outside sources.

![Output](/img/posts/2016-11-26-getting-to-know-consul//9.png)

In the example above you may of noticed the health check doing a simple http request on `localhost:4000/ping`. These health checks can be simply created in blocks of JSON and stored on the node.

A basic example of this would be the following:

```
echo '{"check": {"name": "ping",
  "script": "ping -c1 google.com >/dev/null", "interval": "30s"}}' \
  >/etc/consul.d/ping.json
```

Once the file is on disk in `/etc/consul.d` you can simply run `consul reload` to have the new check initiated.

You can read more about Health Checks [Here](https://www.consul.io/intro/getting-started/checks.html).

### Conclusion

Consul is a fantastic tool when it comes to service discovery in a micro-services architecture. I look forward to creating a larger scale demo in the near future with NodeJS and MongoDB to illustrate the tool further and learn more about it's inner workings.
