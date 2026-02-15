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

## A Bit of History

Confitura 2017 was the 11th edition of this Java/JVM/programming conference in Poland. When I realized the event has been around for so long, I checked my notes and found that I have only missed the first two editions. This was my 9th Confitura: twice as an organizer, once as a speaker ([2nd best talk](https://www.youtube.com/watch?v=4Dr3dTTxiEk) in 2015, yay!), once as a volunteer, and the rest as a regular attendee.

Confitura started in 2007 as Javarsovia. However, due to a mysterious company trying to ban the word, it was renamed to its current brand. It has grown from a single-track event to the biggest free-to-attend (with a small donation to charity) JVM conference in the country, featuring 5 tracks and over thirty speakers.

## The Talks

When I reached the conference venue, I was surprised by the very short queues at the registration desks. Everything went smoothly—perhaps thanks to the use of QR codes as e-tickets. Within a minute, I had my bag, which even included a trendy fidget spinner toy! 🙂

**Keep IT clean: mid-sized building blocks and hexagonal architecture in real life by [Jakub Narbrdalik](https://twitter.com/jnabrdalik)**

Every time you attend one of Jakub’s talks, you can be sure your time won’t be wasted. This time was no different: a very good presentation on limiting class visibility as a step toward cleaner architecture, better testability, and happier developers. He also covered packages and subpackages, managing the visibility of Spring beans, and a high-level look at CQRS architecture. With Kuba’s talk, Confitura started with some very "meaty" technical content. [Link to the slides](https://jakubn.gitlab.io/keepitclean/#1)

**Studium przypadku wystarczająco dobrej architektury by [Jakub Kubryński](https://twitter.com/jkubrynski)**

The story of a startup and a CEO who needs to balance the eagerness to refactor with the cost of doing so. It included many useful tips on how to apply a "good enough" approach to a project, what to automate, and which metrics to measure to understand what is happening in your system. Kuba explained how they ensure microservices can communicate with each other ([Spring Cloud Contract](http://cloud.spring.io/spring-cloud-contract/spring-cloud-contract.html)), why end-to-end tests should only cover the most critical paths, and why your users are often your best testers. [Link to the slides](https://speakerdeck.com/jkubrynski/case-study-of-the-good-enough-architecture)

**Reactive web services by [Kamil Szymański](https://twitter.com/kszdev)**

A live coding demo showing how to introduce a reactive approach to your project without using too many external libraries (using only the latest Spring and Hibernate). I respect all speakers who are brave enough to write code during a talk, and Kamil did it well, with only one or two minor pauses. He covered Mono and Flux from Reactive Spring as well as features from the latest Hibernate.

**Evolving Architecture by [Daniel Pokusa](https://twitter.com/dpokusa)**

A talk about the career of a software developer trying to understand what makes a "good" architecture and how to build such a system, while making some costly mistakes along the way. It was a very good talk, especially for junior developers; those more experienced might find some of the material familiar. Daniel mentioned several excellent books to read; I personally added the free [Cloud Design Patterns](https://www.microsoft.com/en-us/download/details.aspx?id=420) from Microsoft to my reading list. [Link to the slides](https://www.slideshare.net/danielpokusa/evolving-architecture-4-confitura-2017)

**Visa Developer Center by Tomasz Cisowski**

This was a sponsored presentation, so I didn’t expect too much, but the topic was interesting. I have some experience integrating payment providers and wanted to see what Visa has to offer. In short: visit [developer.visa.com](https://developer.visa.com/#) to check their documentation; they have many different APIs to experiment with.

**Testy wydajności z Gatlingiem by [Andrzej Ludwikowski](https://twitter.com/aludwikowski)**

My colleague from SoftwareMill retold the familiar story that measuring performance is hard. However, with [Gatling](http://gatling.io/)—a tool written in Scala, Akka, and Netty—performing these tests using its elegant DSL should no longer be a pain (we all remember the "great" UX of JMeter). [Link to the slides](https://www.slideshare.net/AndrzejLudwikowski/performance-tests-with-gatling-77489992)

**7 things which you should care about before release your code to production by Mateusz Dyminski**

A solid talk about things like API versioning (especially important for mobile app backends), database migration versioning, and learning how to profile your code *before* you go to production. Lots of useful practical advice! [Link to the slides](https://github.com/mateuszdyminski/7things-java)

**Plantacje programistów – kolonializm XXI wieku by Wojciech Seliga**

I skipped this to do some networking instead, but I had seen this talk at GeeCON earlier this year. Wojtek argued that being "just a coder" is no longer enough; we should strive to understand the business better. We need to be more than someone who just writes code (which is becoming a commodity) and instead understand the product and the organization to bring extra value. He suggested that instead of just software engineering, we should focus on "product engineering," which involves user experience, market research, data analysis, and more. It was a great talk with many interesting insights. [Link to the slides](https://www.slideshare.net/wseliga/developer-plantations-colonialism-of-xxi-century-geecon-2017)

## After-party

I cannot emphasize this enough: after-parties are at least one-third of the value of any technical event. If you have the chance to attend, **never** hesitate. You will meet speakers, organizers, JUG leaders, and many other fascinating people. You can talk to them about anything: their presentations, their open-source projects, or off-topic subjects like cars, kids, Lego sets, or even the strangest place you've ever pushed a Git commit from.

The atmosphere is relaxed, and since no one is rushing to the next session, networking is easy. Just grab a drink and join a group. "Spoina," the party after Confitura, was great as usual. This time, we had [an entire barge](http://miejsce.to/) to ourselves in a very atmospheric location, with good food, plenty of drinks, and no loud music to drown out the conversation.

![Spoina Party]({{site.baseurl}}/images/miejsce-to.jpg)

Of course, we didn’t finish there and eventually moved to the city center for the next phase of socializing! 🙂

## Summary

As usual, Confitura delivered: great talks and even greater people. I saw many old friends and met some new faces too. The old venue (with its upgraded air conditioning) held up surprisingly well. Honestly, aside from there being too many interesting talks to attend and the breaks being too short to talk to everyone I wanted, I couldn't find any drawbacks. If you are into programming, this event is definitely for you.

*Credits: photos by [@bartekzdanowski](https://twitter.com/bartekzdanowski) and [miejsce.to](http://miejsce.to/)*