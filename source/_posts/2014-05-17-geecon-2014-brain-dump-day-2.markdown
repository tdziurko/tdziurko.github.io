---
author: Tomasz Dziurko
comments: true
date: 2014-05-17 21:31:25+00:00
layout: post
slug: geecon-2014-brain-dump-day-2
title: GeeCON 2014 – Brain dump, Day 2
wordpress_id: 1923
categories:
- Conferences
tags:
- GeeCON
---

This is the second part of my notes from [GeeCON 2014](http://2014.geecon.org/) conference.

**Hazem Saleh - Jasmine Automated Tests for JavaScript**



	
  * Testing asynchronous code with [Jasmine](http://jasmine.github.io/2.0/introduction.html) looks really ugly. [Michał Ostruszka](http://michalostruszka.pl/) suggests using [Mocha](http://visionmedia.github.io/mocha/) for such use cases.

	
  * [Istanbul](https://github.com/gotwarlost/istanbul) - code coverage for Java Script


**Sam Newman - Deploying And Testing Microservices**



	
  * [Technology Radar](http://www.thoughtworks.com/radar/#/) from ThoughtWorks

	
  * Book ["Building Microservices"](http://shop.oreilly.com/product/0636920033158.do) by Sam Newman, currently in early version

	
  * It's worth to follow [Sam Newman](https://twitter.com/samnewman) on Twitter

	
  * [Mountebank](http://www.mbtest.org/) - tool to provide test doubles using various protocols (http, https, etc)

	
  * "Get good at releasing services independently"

	
  * Last tests should be performed on production just after the release, but make sure that it do not affect your business

	
  * ["Consumer Driven Contracts"](http://martinfowler.com/articles/consumerDrivenContracts.html) article by Martin Fowler

	
  * [Pact](https://github.com/realestate-com-au/pact) - tool enabling consumer driven contract testing

	
  * [Packer](http://www.packer.io/) - tool for creating identical images for multiple platforms

	
  * [Ansible](http://www.ansible.com/home) - tool to automate app deployment, configuration management, etc., [further reading](http://red-badger.com/blog/2013/06/29/ansible/)

	
  * One service per machine

	
    * pros: easy to reason about (who to blame when CPU is 100%)

	
    * cons: cost and management overhead




	
  * [Docker](https://www.docker.io/) - virtualized container allowing to automate deployment of any application

	
    * "not production ready"




	
  * "Deploy one thing at a time"

	
  * [DbDeploy](http://dbdeploy.com/) - Database Change Management tool

	
  * "You can't expect to change your database as often as you change your software"

	
  * "It's easier to manage one service with two versions of API"


<!-- more -->

**Michał Ostruszka - SRP applied - building modern web applications**



	
  * [gulp](http://gulpjs.com/), [grunt](http://gruntjs.com/) (more popular one) - build systems for frontend/JS

	
  * [yeoman](http://yeoman.io/), [lineman](http://www.linemanjs.com/) - tools to generate bootstrap of frontend part of your application using e.g. AngularJS, Ember, Backbone, etc.


**Peter Lawrey - Micro-second latency logging, persistence, IPC and more**



	
  * [OpenHFT](http://vanillajava.blogspot.com/2013/07/openhft-java-lang-project.html) (High Frequency Trading)

	
  * [Java Chronicle](https://github.com/OpenHFT/Java-Chronicle) project - persisted low latency messaging library

	
  * [HugeCollections](https://github.com/OpenHFT/HugeCollections) - Huge Collections for Java using efficient off heap storage

	
  * [SharedHashMap](http://www.fasterj.com/articles/sharedhashmap1a.shtml), another [link](https://github.com/OpenHFT/HugeCollections/wiki/When-to-use-SharedHashMap):

	
    * GC-free implementation of hash map

	
    * persistent, stored in the file

	
    * shared between processes

	
    * 1/5th of a size of corresponding HashMap in the memory





**Kevlin Henney - Seven Ineffective Coding Habits of Many Java Programmers**



	
  * Book ["97 Things Every Programmer Should Know"](https://www.goodreads.com/book/show/7003902-97-things-every-programmer-should-know)

	
  * Comments in your code are sign of defeat, you failed in explaining your intentions using the code itself

	
  * Max. column width: 80 chars, we shouldn't be forced to move head while reading

	
  * Keep method arguments in one line or each argument in a new line

	
  * Why do we add "Exception" to the exception classes? Do we add "Class" at the end of each class name?


**Kevlin Henney - Worse Is Better, for Better or for Worse**



	
  * Stop over-promising

	
  * Concentrate on something you can complete

	
  * "You learn by finishing things"

	
  * Book ["101 Things I Learned in Architecture School"](101%20Things I Learned in Architecture School), worth to read also from the perspective of developer/architect

	
  * Book ["The Laws of Simplicity: Design, Technology, Business, Life"](https://www.goodreads.com/book/show/225111.The_Laws_of_Simplicity)

	
  * Small off-topic: Kevlin gave one of the best closing keynotes I've ever seen titled "Cool Code". You can watch video [here](http://vimeo.com/44792649). Really entertaining talk!


That's all I've noted from second dat at GeeCON 2014. Stay tuned for the 3rd part of my brain dump.
