---
author: Tomasz Dziurko
comments: true
date: 2009-05-09 11:26:22+00:00
layout: post
slug: eclipse-and-problem-with-auto-generated-equals
title: Eclipse and problem with auto generated equals()
wordpress_id: 123
categories:
- How-to
- Java
tags:
- Eclipse
- Hibernate
- How-to
- IDE
---

Lately my colleague from job encountered following problem:

JSF table <rich:dataTable> was using HashMap to represent data. This Map contained objects collected from the database. After saving new objects to Db and table refresh it showed objects completely different from what we expected. After few hours of searching for a bug in our code or in JSF code it appeared that problem lied in other place.

Eclipse generates equals method checking if classes of compared objects are the same:

[java]
...
іf (getClass() != obϳ.getClass()) {
    return fаlse;
}
...
[/java]

ant such behavior combined with Hibernate can cause some problems because very often while loading data from Db instead of object of our class Hibernate creates proxy object extending base class we are loading. And then equals method comparing getClass() returns false causing strange HashMap behavior. Solution is very simple: instead of getClass() you should use **instanceof**.

Additionally, as I read in comment on [Proxorkut blog](http://www.proxorkut.info/liskovs-substitution-principle-equals-and-hibernate-proxies), if we use lazy loading in entity classes we should use getXXX() methods in equals() and hashcode() to get variable values because if we try to get them directly from the fields, Hibernate might not initialize them before we try to use them.
