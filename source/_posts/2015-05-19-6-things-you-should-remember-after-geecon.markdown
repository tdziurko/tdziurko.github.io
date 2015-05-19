---
layout: post
title: 6 things you should remember after GeeCON 2015
date: 2015-05-19 22:00:10:11 +0200
comments: true
categories: 
- Conferences
- Review
- GeeCON
keywords: GeeCON, Microservices, conference
description: How to design good REST API which is easy to use
---

1200 developers, 75 speakers, 80 talks and 6 important things to remember

<!-- more -->

GeeCON 2015 is over so it is a good time for a summary of this great conference. Below list of my reflections during and just after this great event.

{% img center /images/blog/2015/05/geecon_2015.png 800 800 %}

### 1. Microservices are ultra fancy now

There was no more popular topic than microservices in this years’ talks. And of course we could hear see that “Microservices are the way to go”, “Microservices will help you build scalable systems”, 
“Microservices will help you this, Microservices will help you that”. 

People are in love with this, not so new but currently very hot, approach. As a developer working in microservices environment for almost a year I see many disadvantages, potential problems and issues when applying this approach. 
Especially when idea is introduced without careful planning and checking if company (infrastructure, codebase and especially people) 
is ready for such fundamental change. That is why I am rather sceptical when looking at this, often mindless, “let’s built everything using Microservices” movement.

### 2. Microservices architecture definitely will be overused and/or misused in the near future

In a 2-3 years I expect some talks during conferences with titles like “Migrating from microservices to more monolithic architecture”, “Microservices - what went wrong” or “Getting your project out of microservices hell”. 
People tend to apply things they’ve heard during conferences without considering their own context. In my opinion probably most of our projects will never reach a moment when you should make a decision to split system into 
multiple microservices. Monolith is a simple approach that will be enough for small to medium teams and projects/products.

But worst decision people could make is starting a new project using microservices, especially when your team is new to this approach, domain is new and bounded contexts are very vaguely defined. This is a very quick way leading to project failure.
Unfortunately developers tend to forget that most of their projects are pretty standard and standard problems are best solved by standard approaches.

### 3. PostgreSQL might have all features you need when considering MongoDB

Some time ago many people tend to overuse MongoDB because it promised to solve all our problems with persisting data and scaling DB servers. Of course in many cases it didn't end too well. Moreover MongoDB recently got bad press
(examples: [Stale Reads in MongoDB](https://aphyr.com/posts/322-call-me-maybe-mongodb-stale-reads) or [Call me maybe](https://aphyr.com/posts/284-call-me-maybe-mongodb)). But it looks like potential solution came from completely different direction. 

One of the last releases of PostgreSQL (v. 9.4) introduced support to JSON Binary (more details [here](http://www.infoq.com/news/2014/05/postgresql-9-4)) so it is really worth to check if you really need anything from MongoDB that 
is not provided in proven, well-known traditional RDBMS like PostgreSQL.

### 4. It’s all about People
There are many IT events here in Poland but GeeCON stands out as one which really cares about socialising, making new friends and meeting old ones. I’ve always loved those discussions about IT and non-IT topics over a glass of 
cold beer. And this year was no different. Talking with other developers helps when deciding what technology is worth trying and which one we should avoid at all costs. It also helps to filter “marketing bullshit” when choosing 
new place to work. Some companies promote themselves as great place to be, but quite often talking with some current or former employees could change your opinion significantly. Especially after a few beers, people are more chatty and honest :)

### 5. Remote work is almost here
You can call me biased, but a few years ago only [SoftwareMill](http://SoftwareMill.com) offered full-time remote work for Java developers. Now there are some other options, still not too many, still not 100% remote, still not in a company which was created and is run with remoteness in mind like SoftwareMill, 
but change is visible. As Scott Berkun said during his closing keynote “everything you do in front of the screen, can be dome remotely" and world will be slowly moving in this direction.

And of course, ability to work remotely and company culture will play more and more important role when changing jobs so future looks promising.

### 6. You can't know everything

Three or four years ago if you knew Java Persistence API, one front-end framework (like Spring MVC, Apache Wicket, etc.) and dependency injection (Spring or CDI) you could consider yourself as a pretty good developer. 

Nowadays everything is much more complicated: there is a JavaScript huge popularity growth (not to mention CoffeeScript or TypeScript), AngularJS and whole JS ecosystem with constantly growing number of build tools, 
libraries, frameworks or micro frameworks. We have devOps stuff (Ansible, Puppet, etc.), monitoring and logs aggregation (logstash, ElasticSearch, Kibana), many JVM languages in one project (Java + testing with Spock and Groovy most often), 
growing Scala ecosystem, reactive applications, functional programming and so on, and so on...
 
We are living in hard times for any developer who wants to be up to date with all this stuff that happens around IT and programming. That's why it's very important to consciously choose only a few areas to follow regularly and invest most of your time there. 
Other stuff can wait until it is needed, _lazy val_ was invented for a reason :)

``` scala
    lazy val mySkillsInAreaX = learnSkillsInAreaX()
``` 

Your free time is limited (unless you are no-life person without family, friends or even a pet), so you should spend it with care and with some long-term plan.
 

