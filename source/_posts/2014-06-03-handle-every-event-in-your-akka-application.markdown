---
layout: post
title: "Handle every event in your Akka application"
date: 2014-06-03 09:48:54 +0200
comments: true
categories: 
- Scala
- Akka
keywords: Scala, Akka, events, unhandled events
description: Akka based solution to make sure that every unhandled event will be noticed in Scala or Java application
---

Event here, event there, events flying everywhere. Post about checking that every Akka event will finally find its home

<!-- more -->

{% img center /images/blog/2014/06/akka-logo.png %}


[Akka](http://akka.io) and reactive, event-based applications are new approach to creating software. We are using Akka pretty intensively 
in our current Scala-based project. Events fit our use cases especially well as we are communicating with external API which might be slow. 
This could damage user experience when handled using traditional synchronous approach. But luckily, our requests can be performed 
asynchronously so passing them to Actor seemed a good idea.
 
### When things get ouf of control
  
But while being cool and very useful, events can still hurt project when handled by inexperienced hands. Asynchronous nature makes 
application flow hard to understand at first glance. And each time you add a new actor or event type to your system, probability 
that you will forget to handle something properly increases. 
  
Let's look at the example class, this is an actor handling events associated with Image tags and comments:

                                                 
``` scala

class YourActor extends Actor {
    override def receive = {
        case event: ImageTagged =>
          doSomething()
        case event: OtherImageTaggedByFriend =>
          doSomething2()
        case event: MostMotedUserImage =>
          doSomething3()
        case event: MostCommentedFriendImageChosen =>
          doSomething4()
      }
}

```

and when you add next event, let's say _MostLikedFriendImage_ you can easily forget to add handler case section in actor,
especially if there is more than one actor listening for this type of event.

### DRY violating solution

There is one simple solution that will allow to detect forgotten handlers. We can add case _ to each actor:

``` scala

class YourActor extends Actor {
    override def receive = {
        case event: ImageTagged =>
          doSomething()
        case event: OtherImageTaggedByFriend =>
          doSomething2()
        case event: MostMotedUserImage =>
          doSomething3()
        case event: MostCommentedFriendImageChosen =>
          doSomething4()
        case event: _ : 
          logger.error("Received unknown event " + event.getClass.toString)
      }
}

```

And while it looks pretty ok for one or two actors, adding same code fragment to multiple actors is troublesome and violates DRY
principle. But, what is most dangerous, someone in your team could forget to add it (as someone said _“Every manual task that can be forgotten, will be forgotten”_).
So maybe we should pursue better solution?

### React on ANY unhandled event

{% img center /images/blog/2014/06/event-shall-not-unhandled.jpg %}

Luckily, we are not stuck with our error-prone approach. When actor can not handle event that was passed to him _UnhandledMessage_ is
raised and published to ActorSystem's EventStream.

So to handle every forgotten event we could create listener and subscribe it to EventStream:
   
``` scala

class UnhandledMessageListener extends Actor {

  val logger = LoggerFactory.getLogger(getClass)


  override def receive = {
    case message: UnhandledMessage =>
      logger.error(s"CRITICAL! No actors found for message ${message.getMessage}"))
      
      if (!Environment.isProduction) {
        // Fail fast, fail LOUD
        logger.error("Shutting application down")
        System.exit(-1)
      }
  }
}
   
```

And subscribing code fragment:

``` scala Subscribe logic

 val actorSystem = ActorSystem.create("projectActorSystem")
  
 val listener = actorSystem.actorOf(Props(new UnhandledMessageListener()))
 actorSystem.eventStream.subscribe(listener, classOf[UnhandledMessage])

```

and that's it. Now every time there is an event that wasn't handled by actor, we will know about it, especially when application 
is deployed in a non-production environment :)
   
   
   







                                                 
                                                 