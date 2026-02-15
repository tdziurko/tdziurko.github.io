---
layout: post
title: Confitura 2017 – greatness delivered
description: Last weekend, I attended Confitura 2017. This year's conference had about 1,400 attendees, 5 tracks, and 35 presentations — there was something interesting for everyone to learn. This is my review.
date: 2017-07-04
author: tomek
image: '/images/confitura-entrance.jpg'
tags: [Conferences, Java, Confitura]
tags_color: '#aeb85f'
---

Last weekend, I attended Confitura 2017. This year's conference had about 1,400 attendees, 5 tracks, and 35 presentations; everyone could find something interesting to listen to and learn from. This post is a review of that great event.

## A bit of history

Confitura 2017 was the 11th edition of this Java/JVM/programming conference in Poland. When I realized the event has been around for so long, I checked my notes and found that I have only missed the first two editions. This was my 9th Confitura: twice as an organizer, once as a speaker ([2nd best talk](https://www.youtube.com/watch?v=4Dr3dTTxiEk) in 2015, yay!), once as a volunteer, and the rest as a regular attendee. Confitura started in 2007 as a Javarsovia but with [a mysterious company trying to ban Java word](../../04/java-word-will-be-banned/index.html), it was renamed to its current brand, growing from a single track event to the biggest free-to-attend (only a bit for the charity) JVM conference in our country with 5 tracks and over thirty speakers.

## Talks

When I reached conference venue I was surprised by very short queues in tsuhe front of registration desks. Everything went smoothly, maybe thanks to QR codes as e-tickets? After less than a minute I got my bag, even with a trendy fidget spinner toy 🙂

**Keep IT clean: mid-sized building blocks and hexagonal architecture in real life by [Jakub Narbrdalik](https://twitter.com/jnabrdalik)**

Every time you attend Jakub’s talk you could be sure that time spent there won’t be wasted. And this time it was no different: very good presentation about limiting classes visibility as a step to create cleaner architecture, better testability and happier developers. He also told covered packages and subpackages, managing visibility of Spring beans and a high-level look at CQRS architecture. With Kuba’s talk Confitura started with a very tasty technical meat. Slides: [link](https://jakubn.gitlab.io/keepitclean/#1)

**Studium przypadku wystarczająco dobrej architektury by [Jakub Kubryński](https://twitter.com/jkubrynski)**

Story of a startup and a CEO who needs to balance eagerness to refactor and cost of doing refactors in his project, many useful tips how to apply “good enough” approach to your project, where to automate and what things to measure to understand what is going in your system. Kuba told a bit how they make sure that all microservices could communicate with each other ([Spring Cloud Contract](http://cloud.spring.io/spring-cloud-contract/spring-cloud-contract.html)), why end to end tests should only cover most critical paths in your application and why users are your best testers. Slides: [link](https://speakerdeck.com/jkubrynski/case-study-of-the-good-enough-architecture)

**Reactive web services by [Kamil Szymański](https://twitter.com/kszdev)**

Live coding demo showing how to introduce reactive approach to your project without using many external libraries (only latest Spring and latest Hibernate). I respect all speakers that are able to write code during their talk and Kamil did it well, with only one or two longer pauses. There was a bit about Mono and Flux from Reactive Spring and a bit from latest Hibernate.

**Evolving Architecture by [Daniel Pokusa](https://twitter.com/dpokusa)**

Talk about career of Software Developer who tried to understand what is a good architecture and how to create such system making some costly mistakes along the way. Very good talk, especially for junior developers, those more experienced might find some stuff rather obvious. Daniel mentioned many very good books you should definitely read, personally I have added one (free [Cloud Design Patterns](https://www.microsoft.com/en-us/download/details.aspx?id=420) from Microsoft) to my reading list too. Slides: [link](https://www.slideshare.net/danielpokusa/evolving-architecture-4-confitura-2017)

**Visa Developer Center by Tomasz Cisowski**

It was a sponsored presentation, so I didn’t expect too much but topic was interesting, I have some experience with integrating payment providers and wanted to learn what Visa has to offer in this area. In short: go to [https://developer.visa.com/#](https://developer.visa.com/#) and check their docs, they have many different APIs to play with.

**Testy wydajności z Gatlingiem by [Andrzej Ludwikowski](https://twitter.com/aludwikowski)**

My colleague from SoftwareMill told a known story that measuring performance is hard, but with [Gatling](http://gatling.io/), tool written in Scala, Akka and Netty at least performing such tests using its fine DSL should no longer be a pain (JMeter, we all remember your great UX). Slides: [link](https://www.slideshare.net/AndrzejLudwikowski/performance-tests-with-gatling-77489992)

**7 things which you should care about before release your code to production by Mateusz Dyminski**

Good talk about things like API versioning (especially important in backends for mobile apps), DB changes versioning, checking how to profile at least once before you go to the prod and many other useful stuff to cover before actual release. Many useful stuff! Slides: [link](https://github.com/mateuszdyminski/7things-java)

**Plantacje programistów – kolonializm XXI wieku by Wojciech Seliga**

I skipped it and choose networking instead, but I have seen this talk at GeeCON this year. Wojtek told that being only a coder is not enough and we should try to understand business better, become someone not only writing a code (which will become a commodity soon), but also try to understand product, understand organization you are in to bring something extra. He said that instead of software engineering we should try to learn product engineering with much broader scope involving areas like user experience, market research, data analysis and many many more. Very good talk with many interesting insights. Slides: [link](https://www.slideshare.net/wseliga/developer-plantations-colonialism-of-xxi-century-geecon-2017)

## After-party

I can not underline this enough: after-parties are at least 1/3 of any technical event, so if you have a chance to attend one, do not **EVER** hesitate. You will meet speakers, organizers, JUG leaders and many, many other very interesting people. You could talk to them about anything: their talk, their open-source project or more off-topic like cars, kids, Lego sets or even the strangest place from you pushed a commit to Git repo. Atmosphere is more relaxed, no one is in a hurry for the next presentation so networking is really, really easy. Just grab your juice/beer/water/whatever and join any group discussing about something you think might be interesting. And Spoina, party after Confitura, was great as usual. This time we had [a whole barge](http://miejsce.to/) for us in a very climatic location, good food, plenty of different drinks and no loud music.

![Spoina Party]({{site.baseurl}}/images/miejsce-to.jpg)

Of course we didn’t finish there and landed in the City center for a next phase of integration 🙂

## Summary

Confitura as usual delivered: great talks and even greater people. Many old friends and some new faces too. Old venue (with upgraded air conditioning) did surprisingly well, so actually except too many interesting talks I could not attend and too short breaks to talk with everyone I wanted, I could not finy any drawbacks. If you are into programming, this event is definitely for you.

*Credits: photos by [@bartekzdanowski](https://twitter.com/bartekzdanowski) and [miejsce.to](http://miejsce.to/)*