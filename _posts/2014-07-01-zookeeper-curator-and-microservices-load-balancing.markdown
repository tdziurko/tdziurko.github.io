---
layout: post
title: "ZooKeeper, Curator and how microservices load balancing works"
description: Service discovery and out of the box load balancing of microservices with Apache ZooKeeper and Curator
date: 2014-07-01
author: tomek
image: '/images/zookeeper-cover.jpg'
tags: [Programming, Java]
tags_color: '#5fb05f'
redirect_from:
  - /2014/07/zookeeper-curator-and-microservices-load-balancing/
---

How Zookeeper makes sure that every worker happily gets some stuff to do from job delegating manager

[Apache ZooKeeper](http://zookeeper.apache.org/) is a tool to register, manage and discover services working on different machines. It is an indispensable member in the technology stack when we have to deal with a distributed system with many nodes which need to know where their dependencies are launched.

![Apache ZooKeeper logo](/images/zookeeper-logo.png)

However ZooKeeper is pretty low level and even standard use-cases require many lines of code. That is why [Apache Curator](http://curator.apache.org/) was born – a much more friendly and easier to use wrapper library over ZooKeeper. Using Curator we can deliver more with less code and in a much cleaner way.

![Apache Curator logo](/images/curator-logo.png)

> “Guava is to Java what Curator is to ZooKeeper” – Patrick Hunt, ZooKeeper committer

## Load balancing microservices with ZooKeeper

We are accustomed to the situation when there is a load balancer deployed in front of our application. Its role is to make sure that every single node gets more or less the same amount of traffic.

In a microservices world the situation is the same but we have a significant advantage over a monolithic approach: when we need to scale an application we don’t have to duplicate the whole system and deploy it on a server powerful enough to run it smoothly. We can concentrate only on a small module that needs to be scaled so the cost of scaling is much lower (both in terms of server cost and development needed to prepare module for deployment in many instances).

But as our system grows, we end up with many scaled modules, each of them requiring a separate load balancer. This seems troublesome as our infrastructure is very complex even without them. Luckily if we use ZooKeeper as a service orchestration and discovery tool we can use built-in load balancing capability without introducing any additional complexity into our microservice architecture.

To present how out of the box load-balancing works in ZooKeeper we need two services: a worker which will be deployed multiple times and a manager delegating tasks to registered workers.

## Simple worker

Let’s start with creating a simple worker that will listen on a given port and return some result when asked to perform its job. To implement this tiny microservice we will use Groovy, [Undertow](http://undertow.io/) light-weight servlet container and of course ZooKeeper and Curator.

Our worker will consist of one, small class with a _main_ method that does three things:

```groovy
class Main {

    static final void main(String[] args) {
        // Step 1: Extract name and port number to launch worker
        // Step 2: Configure and start Rest server on given port
        // Step 3: Register worker in ZooKeeper
    }
}
```

For brevity I will omit steps 1 and 2 here, you can check the complete source code on [GitHub project](https://github.com/tdziurko/microservices-load-balancing).

Our worker will have only a single endpoint GET /work which returns a response with the name of the called worker:

```groovy
@Path("/")
class Main {

    @GET
    @Path("/work")
    public String work() {
        String response = "Work done by $workerName"
        println response
        return response
    }

}
```

Step 3: _Register worker in ZooKeeper_ is the most interesting here so I will explain it in more detail:

```groovy
private static void registerInZookeeper(int port) {
        CuratorFramework curatorFramework = CuratorFrameworkFactory.newClient("localhost:2181", new RetryNTimes(5, 1000))
    curatorFramework.start()
    ServiceInstance<Void> serviceInstance = ServiceInstance.builder()
        .uriSpec(new UriSpec("{scheme}://{address}:{port}"))
        .address('localhost')
        .port(port)
        .name("worker")
        .build()

    ServiceDiscoveryBuilder.builder(Void)
        .basePath("load-balancing-example")
        .client(curatorFramework)
        .thisInstance(serviceInstance)
        .build()
        .start()
}
```

- Lines 2-3: we create and start a CuratorFramework client wrapping all operations we want to perform on ZooKeeper instance. For simplicity we use localhost with default port (in general it should be the URL to the running instance of ZooKeeper)
- Lines 4-9 create ServiceInstance representing our worker. We pass all “contact details” required to call this worker from other microservices
- Lines 11-16 register our instance in a ZooKeeper represented by CuratorFramework client.

## Starting worker

Worker is now ready so we can create a fat jar (with Gradle _fatJar_ task) and then launch it using

```bash
java -jar simple-worker/build/libs/simple-worker-1.0-shadow.jar Worker_1 18005
```

Before starting worker remember that you need a ZooKeeper instance running on the default 2181 port!

To check that worker is running you should open the browser on [http://localhost:18005/work](http://localhost:18005/work) and see “Work done by Worker_1” text there. To verify that worker has properly registered itself in ZooKeeper, launch the command line client:

```bash
cd <ZK_ROOT>/bin
./zkCli.sh
```

and then execute the _ls_ command to see one node registered under _/load-balancing-example/worker_ path:

```text
[zk: localhost:2181(CONNECTED) 1] ls /load-balancing-example/worker <enter>
[f69545e8-8466-40c0-93e9-f493eb7496b4]
```

## Simple manager

Now, as we have a worker listening on _/work_ for requests, we can create a simple manager service delegating tasks to its subordinates. The main method looks quite similar to the one in the simple-worker project, the main difference is that we do not register in ZooKeeper, we only create _ServiceProvider_ whose role is to (surprise, surprise) provide us with an instance of a worker. So the basic workflow is:

1. Wait for requests on _/delegate_
2. Get an instance of the worker service from ZooKeeper’s ServiceProvider
3. Call worker’s _/work_ and return results of its execution

To create ServiceProvider we have to create a CuratorFramework client, connect to ZooKeeper and then fetch ServiceProvider for the service with the given name, _worker_ in our case:

```groovy
CuratorFramework curatorFramework = CuratorFrameworkFactory.newClient("localhost:2181", new RetryNTimes(5, 1000))
curatorFramework.start()

ServiceDiscovery<Void> serviceDiscovery = ServiceDiscoveryBuilder
    .builder(Void)
    .basePath("load-balancing-example")
    .client(curatorFramework).build()
serviceDiscovery.start()

serviceProvider = serviceDiscovery.serviceProviderBuilder().serviceName("worker").build()
serviceProvider.start()
```

- Lines 1-2, create ZooKeeper client, same way as in simple-worker
- Lines 4-8, create ServiceDiscovery which will be able to give us service providers.
- Lines 10-11, create and start ServiceProvider which will be used to fetch a working instance of worker node.

Finally, we need a Rest endpoint on which the manager waits for tasks to delegate:

```groovy
@GET
@Path("/delegate")
public String delegate() {
    def instance = serviceProvider.getInstance()
    String address = instance.buildUriSpec()
    String response = (address + "/work").toURL().getText()
    println response
    return response
}
```

## Starting manager

Manager is ready and after _fatJar_ task we can launch it using

```bash
java -jar simple-manager/build/libs/simple-manager-1.0-shadow.jar 1800
```

To verify it is working we can open [http://localhost:18000/delegate](http://localhost:18000/delegate) in the browser to see message “Work done by Worker_1”.

Manager doesn’t know anything about its workers. The only thing he knows is that the service is registered under specific path in ZooKeeper. And it is that simple no matter if we have multiple workers launched locally or spread across different servers in different countries.

## Out of the box load-balancing with ZooKeeper

Imagine a situation where the manager gets so many tasks from his CEO that he needs more than one worker to delegate the job. In a standard case we will be forced to scale workers and place a load balancer in front of them. But ZooKeeper gives us this feature without any additional work.

Let’s add some more workers listening on different ports:

```bash
java -jar simple-worker/build/libs/simple-worker-1.0-shadow.jar Worker_1 18005 &

java -jar simple-worker/build/libs/simple-worker-1.0-shadow.jar Worker_2 18006 &

java -jar simple-worker/build/libs/simple-worker-1.0-shadow.jar Worker_3 18007 &

java -jar simple-worker/build/libs/simple-worker-1.0-shadow.jar Worker_4 18008 &
```

The trick is that all workers are registered under the same path in ZooKeeper, so when we list nodes under _/load-balancing-example/worker_ we will see four instances:

```text
[zk: localhost:2181(CONNECTED) 0] ls /load-balancing-example/worker  <enter>
[d5bc4eb9-8ebb-4b7c-813e-966a25fdd843, 13de9196-bfeb-4c1a-b632-b8b9969b9c0b, 85cd1387-2be8-4c08-977a-0798017379b1, 9e07bd1d-c615-430c-8dcb-bf228e9b56fc]
```

What is most important here is that to utilize all these four new workers, the manager doesn’t require any changes in the code. We can launch new worker instances as traffic increases or shut them down when there is nothing to do. Manager is decoupled from these actions, it still calls _ServiceProvider_ to get an instance of a worker and passes job to him.

So now when we open [http://localhost:18000/delegate](http://localhost:18000/delegate) and hit refresh several times we will see:

```text
Work done by Worker_1
Work done by Worker_2
Work done by Worker_3
Work done by Worker_4
Work done by Worker_1
Work done by Worker_2
Work done by Worker_3
```

How is it implemented under the hood? By default ServiceProvider uses Round-robin ProviderStrategy implementation which rotates instances available under the given path so each gets some job to do. Of course we can implement our custom strategy if the default mechanism doesn’t suit our needs.

## Summary

That’s all for today. As you can see by using Apache ZooKeeper and Curator we can live without separate load balancers that need to be deployed, monitored and managed. Infrastructure in a microservices architecture is pretty complicated even without them.
