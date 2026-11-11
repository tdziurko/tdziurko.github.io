---
layout: post
title: GeeCON 2014 – Brain dump, Day 3
description: My notes, tools and books from the third, final day of GeeCON 2014
date: 2014-05-18
author: tomek
image: '/images/geecon-2014-day-3-cover.jpg'
tags: [Conferences, Java]
tags_color: '#aeb85f'
---

The last, third post with my notes from the [GeeCON 2014](http://2014.geecon.org/) conference. Pretty short? I am not going to justify myself, but it was the third day and everyone was a bit tired after two intensive days and nights in Cracow.

### Tomasz Kowalczewski – Reactive Java

- Something almost “reactive” is already in JDK8: Completable Future, more reading: [“Java 8: Definitive guide to CompletableFuture”](http://www.nurkiewicz.com/2013/05/java-8-definitive-guide-to.html)
- [RxJava](https://github.com/Netflix/RxJava) has a nice DSL allowing to configure observables in an elegant way
- Project has a [good wiki](https://github.com/Netflix/RxJava/wiki), worth diving deeper
- Observables can be easily converted to Iterator (good for step-by-step migrations)

### Tom Bujok – 33 things you want to do better

- (didn’t attend this one) but there are very good [slides available](https://speakerdeck.com/tombujok/33-things-you-want-to-do-better-3)

### Sander Mak, Paul Bakker – Modular JavaScript

- [CommonJS](http://www.commonjs.org/) (preferred one) and [AMD](https://github.com/amdjs/amdjs-api/blob/master/AMD.md)
- Chuck Norris Stamp Of Approval – [image](http://www.clipartbest.com/clipart-nTXEB7jTB) 😉

### Sandro Mancuso – Crafted Design

- Controller only controls the flow, usually only invokes use_cases
- use_cases, package for logic/features
- Packages by feature
- There were too many diagrams hard to describe in text so you should really check [slides](http://www.slideshare.net/sandromancuso/crafted-design-geecon-2014)

### Michael Feathers – Beyond Error Handling

- Book [“Growing Object-Oriented Software, Guided by Tests”](https://www.goodreads.com/book/show/4268826-growing-object-oriented-software-guided-by-tests)
- We shouldn’t pass nulls inside the application
- “Do not check for stupid everywhere, check for stupid at the door”
