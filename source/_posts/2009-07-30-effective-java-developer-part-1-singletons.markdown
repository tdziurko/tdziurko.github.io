---
author: Tomasz Dziurko
comments: true
date: 2009-07-30 06:46:32+00:00
layout: post
slug: effective-java-developer-part-1-singletons
title: Effective Java developer, part 1 - singletons
keywords: Effective Java, book, review, better code, singleton
wordpress_id: 278
categories:
- Books
- How-to
- Java
tags:
- Books
- How-to
- Java
---

As I said in me previous post, today I am going to start series of posts about the most interesting things from the book "Effective Java". I will try to cover  topics which every Java developer can encounter during common workday.


### Three ways to create singleton


The very, very standard singleton looks like that:

``` java
package pl.tomaszdziurko.effectivejava;
public class LeoMessi {
  public static final LeoMessi INSTANCE = new LeoMessi();
 private LeoMessi() {
   // create only one existing instance of LeoMessi, as he is unique
 }
 public void doMagicWithBall() {
    //do something incredible on the pitch
  }
}

```

<!-- more -->
In the example above we can easily notice that this is a singleton: private constructor and final static filed with only one existing instance of LeoMessi are more than clear signal about it.

**Second way:**

``` java
public class LeoMessi {
  private static final LeoMessi INSTANCE = new LeoMessi();
  private LeoMessi() {
    // create only one existing instance of LeoMessi, as he is unique
  }
  public LeoMessi getInstance() {
    return INSTANCE;
  }
  public void doMagicWithBall() {
    //do something incredible on the pitch
  }
 }
```


Creating singleton as shown above gives us greater flexibility in later changes in the code. We can change internals of getInstance() without breaking our public API, for example we can modify it to return unique instance per thread or per logged user.

Unfortunately both approaches are not reflection-safe and also are not ready for serialization. If we want our singleton remain single after serialization and deserialization we have to overload readResolve() method to return INSTANCE object. And simpe trick to avoid reflection problem is to add few lines in the constructor that throw exception when INSTANCE != null.

**Third way:**

The last one (and the most proper way according to the author) way of creating singleton is doing it with enums:

``` java
  public enum LeoMessi {
  INSTANCE;

  void doMagicWithBall() {
   //do something incredible on the pitch
  }
 }
```


Thanks to using enum we have seralization and reflection problems solved without any additional effort. Everything is delivered by the Java language itself and there is nothing to worry about :)

Personally most often I used approach with private final static field and getInstance() method, but now, after reading Effective Java, I see that enum is very interesting alternative I try to use in the future.
