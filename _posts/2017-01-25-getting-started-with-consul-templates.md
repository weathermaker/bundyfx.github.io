---
layout: post
section-type: post
title: Getting Started with Consul Template
category: Consul
tags: [ 'Consul', 'Automation' ]
---

In this post I want to go through Consul Template and how it can save you tons of time by taking out the guess work or templating files used for configuration of your applications. This post is aimed at people who have not used Consul Template before and are looking for a way to create dynamic configuration files that are easily updatable across large fleets of endpoints.

Contents
=================

* [Intro](#intro)
* [Templates](#templates)
* [Example](#joining-the-cluster)
* [Conclusion](#conclusion)


## Intro

[Consul Template](https://github.com/hashicorp/consul-template) provides a convenient way to populate values from Consul into the file system using the consul-template daemon.

**Why would I want this?**

When it's your job to create Automation within your environment you want configurations within files to *just work* and be able to changed on the fly as needed. An Example of this would be something like a configuration file for an application. Let's take [Datadog](https://app.datadoghq.com) for example. Datadog simply requires that `datadog.conf` *(which resides on disk)* contain information about the specific host such as name, tags and information such as the API Key that will be used for communication with the Datadog API.

You might say, *We can do ahead and populate those values before we deploy out Datadog and everything should be fine, right?*

Well.. What if you want to update a tag within the config file?

Sure! you just have to push your configuration out again which will require your CI to run then move along to your deployment all for the update of a single tag.

Consul templates makes it easy to update information within such files from the data stored within Consul. If you've not worked with Consul or just have no idea what I am talking about, be sure to checkout my previous posts on [Getting to know Consul](https://flynnbundy.com/service-discovery/2016/11/26/getting-to-know-consul.html).

## Templates

If you've worked with creating templates before for applications you may be familiar with something like [jinga2](http://jinja.pocoo.org/docs/2.9/). This concept works great but can sometimes require a large amount of upfront investment to get functional and also can be hard to update rapidly and dynamically. As an example, Ansible does a great job of incorporating *.j2* templates within their configuration manager, making it easy to first get those templates on disk. However this means that the Ansible Configuration become your main source of truth as far as what is currently configured within your environment. When we're talking about Containers and Micro-service based architecture this can sometimes not be fast enough.

There are alternative ways of creating Templates. A few risky ways come to mind such as *grep/sed* with Regular expressions on files that could potentially change format and break this functionality; making it a nightmare for the guy who has to troubleshoot and fix.

This is where Consul Template come in, by allowing templates to be updated with data from within Consul. This works by Consul and Consul Template communicating with each other to advise if anything has been updated. If so, Consul Template should generate a new template based on the updated data within Consul and *(if specified)* run any code that was specified when Consul Template was initiated.

## Example

To get our example started we're going to just run a single instance consul agent like so:

```
docker run -p 8400:8400 -p 8500:8500 -p 8600:53/udp -h node1 progrium/consul -server -bootstrap -ui-dir /ui
```

Once we've got that up and running we should be able to query to Key/Value store and get a 404 *(meaning no data)* error. Nice, Ok let's add some data into our Key/Value store that we're going to use later on.

```
curl -X PUT -d 'service:redis,environment:acceptance' http://localhost:8500/v1/kv/web/redis-datadogtags
```

You should get back a response of **true** once the information has been stored.
Of course to see that information you can *curl* the endpoint like so to return your JSON data.

```
curl http://localhost:8500/v1/kv/web/redis-datadogtags
```

Let's get the binaries for Consul Template now and see how this works. You can download Consul Template for [here](https://releases.hashicorp.com/consul-template/). Once you've got that downloaded and in your path you should be able to access Consul-Template by simply running `consul-template -h` to produce the help contents.

Let's go and setup our *datadog.conf* file ready to be managed by Consul Template. Firstly, Consul Template requires that the file extension be changed to **.ctmpl**.

![ctmpl](/img/posts/2017-01-25-getting-started-with-consul-templates/1.PNG)

Let's also open up the file and make some changes.

Here is the small bit of the file that we're going to be working on in this example:
{% raw %}
```
# Set the host's tags (default: no tags)
tags: "{{key "web/redis-datadogtags"}}"
```
{% endraw %}
We are simply finding the tags section within the *.conf* file and replacing the value with the location of the data that we want to grab from Consul.

Now, let's save that file and head over to our command-line. By running the following command we can tell Consul Template to take a look at the *.ctmpl* file and replace any specified substitution information with actual data from Consul.

```
./consul-template.exe -consul-addr localhost:8500 -template "C:\ProgramData\Datadog\datadog.ctmpl:C:\ProgramData\Datadog\datadog.conf"
```

Once that has been executed you should see your newly created *datadog.conf* within the output path you have specified. In this case its *C:\ProgramData\Datadog\datadog.conf*.

Taking a closer look we can see that the values have been exchanged with the data in Consul as we expected:

```
# Set the host's tags (default: no tags)
tags: 'service:redis,environment:acceptance'
```

You may also notice that your terminal window is still open and unavailable for your commands. This is by design and is part of the way Consul Template works. Consul Template is designed to be run until stopped and allows you to make changes in Consul that will in turn update any of the files that are currently being *'watched'* by Consul Template.

Consul Template takes any number of *-template* commands that can point to any *.ctmpl* file and also watches all files in synchronicity.

Now, Let's make Consul Template Restart our Datadog Service if it notices any changes to this specific item in our Key/Value store.

```
 ./consul-template.exe -consul-addr localhost:8500 -template "C:\ProgramData\Datadog\datadog.ctmpl:C:\ProgramData\Datadog\datadog.conf:powershell.exe restart-service DatadogAgent"
 ```

 If we now update our *"web/redis-datadogtags"* value within our Key/Value store; Consul Template will substitute in the new value into the file being watched and run the command that is specified after the colon. In this case we call PowerShell.exe to restart our DatadogAgent Service.

 This allows the service to re-register with the Datadog endpoint and update any configuration within the web console.

 This is a fairly simple example, however you can learn more about the sort of variables, data and statements that can be used from within Consul Template on their [Github page](https://github.com/hashicorp/consul-template).

## Conclusion

Consul Template is one of those tools that will see a boost when it comes to working with Containers and Micro-services. The General Systems Engineer has probably come across many times they could of used both Consul and Consul Template to save time updating static files on disk of large fleets of servers.

There are many scenarios that come to mind when thinking of this sort of tool. When it comes to the Cloud we want to be able to deploy our nodes out and for our applications and to be able to just work with data that is relevant to that specific application. However, immutable infrastructure makes this challenging. This is where Consul Template really shines. The instance can simply check its tag data to find out more about itself then can ask Consul to fill in any gaps that are required in order to have everything up and running.
