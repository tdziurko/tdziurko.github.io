---
layout: post
title: Lobotomy to your object oriented thinking - "Elegant Objects, volume 1" book review
description: My impressions after reading this popular but quite controversial book about object-oriented programming
date: 2017-07-26
author: tomek
image: '/images/elegant-objects.png'
tags: [Reading, Review]
tags_color: '#477690'
---
> Step one in the transformation of a successful procedural developer into a successful object developer is a lobotomy.
>
> <cite>– David West</cite>


This is the first sentence in ["Elegant Objects, volume 1"](https://www.amazon.com/Elegant-Objects-1-Yegor-Bugayenko/dp/1519166915) book by Yegor Bugayenko and after reading it from cover to cover 
I could not agree more. This book will not leave you neutral, you will either strongly agree or disagree 
with claims stated there, but it is definitely worth your time. It will challenge what you know about programming, 
it will challenge what you think a proper object-oriented design is and it will challenge many old, well-established 
so called "good practices" you have seen during your career. Fasten your belts, move your coffee mug away from your 
keyboard and keep reading.

![Lobotomy to your brain]({{site.baseurl}}/images/homer-brain.jpg)
*Image from [http://simpsons.wikia.com](http://simpsons.wikia.com)*

## Content

"Elegant Objects, vol. 1" in over 200 pages gives you 23 practical tips for developers to write more object-oriented, 
thus more maintainable code. The author uses very interesting allegory by treating every object as a human being 
and splitting these suggestions into a four anthropomorphized chapters: birth, school, employment, and retirement.

**_The Birth_** chapter is rather short and covers object creation, starting from naming strategies through suggestions 
how to keep constructors clean and easy to test. Education and Employment are the core of the book and 
contain paragraphs describing how to build encapsulated, immutable and easy to test objects, how to use interfaces 
with stub implementations to simplify testing without mocking hell. Then we could read some rants about 
"encapsulation by getters/setters", sections about the clear distinction between data structures known from 
procedural languages and well-defined classes and objects from OO programming. Last paragraph Retirement is about 
avoiding nulls, using only checked (yes, yes, no typo here :) ) exceptions and recovering from errors.

Each suggestion is clearly described with code snippets showing bad, and then, proper approaches to solve the discussed topic. There are also links to discussions on author's blog so you could read what other have to say about each suggestion. Most paragraphs are short so the reader could split reading sessions into smaller ones without the need to re-read several pages. Paragraphs are also rather autonomous so you could read them in any order, even if there is a reference to something covered elsewhere, it will be a direct reference to paragraph number where the reader should look for mentioned information.
Everything seemed to be stable – there were Team A and Team B and they were the best places to improve players’ skills. 
Teams C and D paid super high salaries and bonuses for goals, but at the same time they used old approaches far away from 
modern football. Knowing this, every good player could discuss the future with his agent to decide what is the best for him 
at a given point of his career as a professional footballer.

## My Notes

During the reading, I put many sticky notes with sentences worth remembering, below the most important or intriguing ones.

* Avoid names ending with "-er", it suggests that such object is only a collection of procedures that manipulate data 
and not a fully-independent entity that is capable of acting on its own
* Each class should have only one primary constructor with initialization logic there, non-primary (secondary) constructors should only prepare data (params conversion, etc.) and call a primary one.
* Each class should have maximum four properties. Author makes an analogy to coordinates in the universe (x, y, z and time)
* Instantiation should be strictly separated from execution, new operator is allowed only inside secondary constructors
* Every public method must be a part of some interface. Why? See note below.
* Avoid mocking hell by using no mocks, provide default (fake) implementations of all interfaces that could be later used in your tests. This way other developers won't have to setup everything in each test, they could simply use fake implementations that need to be written once.
* Avoid mutable state.

> Mutable objects are an abuse of the entire object-oriented paradigm.

* Avoid temporary coupling between lines: with immutable objects you do not have two separate phases of 
object instantiation and object initialization (first "new", then multiple setters) so we can not 
accidentally put any logic before the object is fully ready to use. The immutable object is either 
non-existing or fully initialized entity ready to work for you, so reordering lines won't have any 
effect on application logic, the same immutable object will be used.

* Show your respect to others by writing code that assumes they are junior programmers. Not by showing 
off and demonstrating your skills but by writing simple and easy to follow code.
* Favor small objects with the well-defined scope and responsibility -> max four public/protected 
methods. If this number goes above five, think carefully as probably this class has more than one responsibility.

> Static methods are like a cancer to your objects.

* You can't trust objects returning null, every time at some point you will have to check for nullability.

As you can see some of them are only a bit more strict rules that we used to apply during our daily job, 
but some are quite radical. If you want to understand them better or read if readers agree or disagree 
with presented approach you could use a link to the discussion on Yegor's blog (e.g. [Don't mock, use fakes](https://www.yegor256.com/2014/09/23/built-in-fake-objects.html)). 
I am not sure that applying all of them is possible, but in my opinion, the more you try the better your code will be.

## Summary

Without any doubt, I am placing "Elegant Objects, vol. 1" next to my other favorite programming books like 
"Clean Coder", "Clean Code', "Software Craftsman" and "The Pragmatic Programmer: From Journeyman to Master" 
(some of them have reviews here on my blog). It shows a very different point of view to object-oriented 
programming and challenges much stuff we all take for granted. It presents something I would call a 
Radical Object-Oriented paradigm that will broaden your horizons and make you think twice before 
following any standard approach. There is second part named ["Elegant Objects, volume 2"](https://www.amazon.com/Elegant-Objects-2-Yegor-Bugayenko/dp/1534908307) 
that I am planning to read as well.