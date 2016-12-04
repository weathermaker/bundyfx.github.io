---
layout: post
section-type: post
title: Dynamic Inventory with Consul and Ansible
category: Ansible
tags: [ 'Ansible', 'Consul' ]
---

In this post I want to go over using Ansible with a dynamic inventory generated with Consul. Ansible is a great tool for Configuration Management and really lives up to its mantra of simplicity. However, when it comes to management of static inventory files it can become messy quite quickly.

Contents
=================

* [Intro](#intro)
* [Consul Cluster](#consul-cluster)
* [Adding Clients](#adding-clients)
* [Joining the Cluster](#joining-the-cluster)
* [Adding Services](#adding-services)
* [Running Ansible](#running-ansible)
* [Dynamic Inventory](#dynamic-inventory)
* [Conclusion ](#conclusion)

### Intro

Inventory in Ansible is a simplistic concept, however as you add more and more *snowflake* type servers in it can become quickly over complicated and cluttered. Dynamic inventory in Ansible allows us to query an endpoint to retrieve inventory data that can be dynamically updated from other sources. This *'endpoint'* can really be anything that is holding information in regards to your nodes such as VMware, AWS or something like Consul.

### Consul Cluster

Since we're going to need a Consul Cluster running we may as well go the quickest and easiest route and use [progrium/consul](https://hub.docker.com/r/progrium/consul/) Docker image.

Spin up the three Consul nodes as described under the **Testing a Consul cluster on a single host** section.

![Output](/img/posts/2016-12-4-dynamic-inventory-with-consul-and-ansible/1.png)

The above picture is taken from [Docker Kitematic](https://github.com/docker/kitematic). This is a simple way to visualize our Containers and give us up-to-date information with constant log tailing.

### Adding Clients

So, once our Consul cluster is up and running we will want to add some clients to it.

For this example we'll just use *centos* which we can get going quickly by doing `docker run --expose=22 -d bundyfx/centos-consul`

You can jump onto this container by running: `docker exec -it {containerID} /bin/bash`

This Container image is based upon this [centos-ssh](https://github.com/jdeathe/centos-ssh) image as it's all setup ready to use with SSH. *(required for ansible)*

It's also had the Consul binaries installed on it, however if you wanted to do this via a fresh centos image you could do as like:

```shell
yum -y install unzip wget
cd /usr/local/bin
wget https://releases.hashicorp.com/consul/0.7.1/consul_0.7.1_linux_amd64.zip
unzip *.zip
```

### Joining the Cluster

Once we have Consul installed we can simply join our cluster as a Client by running:

`consul agent -join 172.17.0.4 -data-dir /tmp/consul -config-dir=/etc/consul.d`

In this case, *172.17.0.4* is one of the *three* nodes within the Consul Cluster. We also need to specify a folder in which Consul will use as its data directory **and** a directory to use for any configurations *(services/checks)*.

![Output](/img/posts/2016-12-4-dynamic-inventory-with-consul-and-ansible/2.png)

You can check the log in Kitematic or by running `docker logs` on any of the nodes in the Consul Cluster to see more information about the new node being registered. Of course you can also run `consul members` to see the list of nodes that are registered.

Let's add in a few more nodes just so we have a bit more data to work with.

![Output](/img/posts/2016-12-4-dynamic-inventory-with-consul-and-ansible/3.png)

Now we have our *three* clients.

### Adding Services

This is all well and good however none of these clients are running any specific Services yet. Let's create a Service on each of them so that we can filter on that in our Ansible playbook's later on.

As you may remember from my previous post on [Getting to know Consul](https://flynnbundy.com/service-discovery/2016/11/26/getting-to-know-consul.html) we went through setting up a Consul Service. For this example let's keep things simple and go with a very similar service.

On one of our client containers let's make a nodejs.json:

```shell
echo '{"service": {"name": "NodeJS", "tags": ["nodejs"], "port": 80}}' \
| tee /etc/consul.d/nodejs.json
```

And this on a different container:

```shell
echo '{"service": {"name": "Redis", "tags": ["redis"], "port": 6379}}' \
| tee /etc/consul.d/redis.json
```

And let's do MongoDB also on our third container:

```shell
echo '{"service": {"name": "MongoDB", "tags": ["mongodb"], "port": 27019}}' \
| tee /etc/consul.d/mongodb.json
```

Now that those configurations are in place we can simply run `consul reload` to trigger an update of our agent. This will look into our configuration directory and load in any services or checks specified.

After our reload, we can hit our cluster to return a list of all services that are defined:

`curl 172.17.0.2:8500/v1/catalog/services`

![Output](/img/posts/2016-12-4-dynamic-inventory-with-consul-and-ansible/4.png)


### Running Ansible

*Little note here*: You should never need to use Configuration Management on Containers to configure... anything.. This is simply an example of how dynamic inventory works on Ansible and Containers are an easy way to demonstrate this functionality.

Like everything else these days, Ansible is available on Docker hub. You can simply download it by running `docker run -it ansible/centos7-ansible`.

Now that we're on our Ansible container let's download the Consul binaries also for use later on. Once we've done that let's use the same command as before and join our Ansible Container to the Consul Cluster. This time however you will want to save your Consul binaries which the `/opt/ansible/ansible/bin/` path for simplicity.

### Dynamic Inventory

So now that we've got everything running as we planned. It's time to go through dynamic inventory with Consul!

The magic glue that ties this all together is the official `consul_io.py` code that is available on the [Ansible repository](https://github.com/ansible/ansible/blob/a3f88eddad772fb0f2e3c1177d1ed08c01e48c48/contrib/inventory/consul_io.py).

Go ahead and clone this repository down. Once you've got it locally in the Ansible Container copy the **consul_io.py** and the **consul.ini** into your **/opt/ansible/ansible/bin** directory.

Crack open the consul.ini file and throw in the name of one of the server nodes within the Consul Cluster.

![Output](/img/posts/2016-12-4-dynamic-inventory-with-consul-and-ansible/5.png)

Before we can run our inventory query we will need to do a quick pip install:

`pip install python-consul`

Once that's done we're good to go.

Simply run the consul_io.py file to see an output taken directly from Consul.

![Output](/img/posts/2016-12-4-dynamic-inventory-with-consul-and-ansible/6.png)

So now we have this data, let's get Ansible to run a command against nodes from our output. We can simply pass in the consul_io.py file as our inventory as specify a role.

![Output](/img/posts/2016-12-4-dynamic-inventory-with-consul-and-ansible/7.png)

This makes it effortless for Ansible to update new nodes that join Consul.

For testing purposes you can use the [Insecure private key](https://github.com/mitchellh/vagrant/blob/master/keys/vagrant) as I have for this demo.

In a production scenario you would also want to setup health checks in that your container's would leave `consul force-leave` the cluster if they failed certain health checks. This way Ansible would not attempt to deploy to them if they are offline.

### Conclusion

Ansible is a great tool for Configuration Management. Coupled with new age tools such as Consul for dynamic inventory and you've got yourself a truly flexible system that requires no manual *touch-ups* to get nodes configured.
