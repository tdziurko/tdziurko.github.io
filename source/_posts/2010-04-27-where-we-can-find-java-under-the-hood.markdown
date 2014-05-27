---
author: Tomasz Dziurko
comments: true
date: 2010-04-27 19:13:01+00:00
layout: post
slug: where-we-can-find-java-under-the-hood
title: Where can we find Java under the hood?
wordpress_id: 305
categories:
- Java
tags:
- Java
---

Browsing today through the blogosphere I encoutered very interesting new: [LinkedIn](http://www.linkedin.com/) is in 99% written in Java. More details can be found in slideshows [here](http://www.slideshare.net/linkedin/linkedins-communication-architecture).  The most suprising thing for me was that they use pure JDBC while we can hear from almost everywhere about power of ORMs. Maybe it's a sign that in applications where we need strict control about what and when is done with database, JDBC is the tool to use?

Going further I began to wonder what technology stacks are used in large portals/services. My finding about LinkedIn encouraged me to thing about technologies used in Polish large internet companies and whether someone use Java or not :)

At this moment I know that:



	
  * [Allegro](http://allegro.pl) (Polish e-Bay) is using Symfony, PHP framework.

	
  * Grono.net: Django and Python

	
  * one of the telecom web services uses Wicket and Java

	
  * shop Merlin.pl is written in Java.


And when we consider foreign portals I read that Twitter was written in Ruby on Rails and later on there were some rumours that after having performance  problems they were thinking about living RoR but how it finally ended, I don't know. All I know for sure that mobile version of MalMart is in Wicket [article](http://www.breakitdownblog.com/apache-wicket-powers-mobile-walmart-com/).
