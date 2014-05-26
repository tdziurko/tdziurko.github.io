---
author: Tomasz Dziurko
comments: true
date: 2012-01-23 18:18:46+00:00
layout: post
slug: google-guava-eventbus-easy-elegant-publisher-subscriber-cases
title: Google Guava EventBus - an easy and elegant way for your publisher - subscriber
  use cases
wordpress_id: 1271
categories:
- Java
tags:
- eventbus
- google
- guava
---

Google Guava in version number 10 introduced new package [eventbus](http://docs.guava-libraries.googlecode.com/git-history/v11.0.1/javadoc/com/google/common/eventbus/package-summary.html) with a few very interesting classes to deal with listener (or publisher - subscriber) use case. Below I present my short introduction to EventBus class and its family.


### Basics


To listen to some events we need a listener class. Such class created in _google-guava-way_ doesn't have to implement any particular interface or extend any specified class. It can be any class with just one required element: a method marked with _@Subscribe_ annotation:

``` java
public class EventListener {

    public int lastMessage = 0;

    @Subscribe
    public void listen(OurTestEvent event) {
        lastMessage = event.getMessage();
    }

    public int getLastMessage() {
        return lastMessage;
    }
}
```

_lastMessage_ property is used in tests below to check if events were received successfully.

And of course we need an event class to send it around:

``` java
public class OurTestEvent {

    private final int message;

    public OurTestEvent(int message) {
        this.message = message;
    }

    public int getMessage() {
        return message;
    }
}
```


#### How it works


The best way to show something in action is to write some tests, so let's see how simple usage of EventBus looks like:

<!-- more -->

``` java
    @Test
    public void shouldReceiveEvent() throws Exception {

        // given
        EventBus eventBus = new EventBus("test");
        EventListener listener = new EventListener();

        eventBus.register(listener);

        // when
        eventBus.post(new OurTestEvent(200));

        // then
        assertThat(listener.getLastMessage()).isEqualTo(200);
    }
```

This test can not be simpler :) We create EventBus instance and listener class instance, then register listener and post new event. And of course this test passes :)


#### MultiListener


Guava also allows to create listener that is reacting for many different events. We just need to annotate many methods with @Subscribe and that's all:

``` java
public class MultipleListener {

    public Integer lastInteger;
    public Long lastLong;

    @Subscribe
    public void listenInteger(Integer event) {
        lastInteger = event;
    }

    @Subscribe
    public void listenLong(Long event) {
        lastLong = event;
    }

    public Integer getLastInteger() {
        return lastInteger;
    }

    public Long getLastLong() {
        return lastLong;
    }
}
```

and a simple example:

``` java
    @Test
    public void shouldReceiveMultipleEvents() throws Exception {

        // given
        EventBus eventBus = new EventBus("test");
        MultipleListener multiListener = new MultipleListener();

        eventBus.register(multiListener);

        // when
        eventBus.post(new Integer(100));
        eventBus.post(new Long(800));

        // then
        assertThat(multiListener.getLastInteger()).isEqualTo(100);
        assertThat(multiListener.getLastLong()).isEqualTo(800L);
    }
```

We don't have to implement multiple interfaces, Guava provides a nice and clean solution with one annotation.


### 




### Beyond the basics


Now let's analyze some more interesting features of EventBus.


#### Dead Event


First unusal thing is a [DeadEvent](http://docs.guava-libraries.googlecode.com/git-history/v11.0.1/javadoc/com/google/common/eventbus/DeadEvent.html) class, predefined event which is fired when we've posted any type of event but no one was there to receive it. To see how it works let's create listener waiting for dead events:

``` java
/**
 * Listener waiting for the event that any message was posted but not delivered to anyone
 */
public class DeadEventListener {

    boolean notDelivered = false;

    @Subscribe
    public void listen(DeadEvent event) {
        notDelivered = true;
    }

    public boolean isNotDelivered() {
        return notDelivered;
    }
}
```

and test case showing how it works:

``` java
    @Test
    public void shouldDetectEventWithoutListeners() throws Exception {

        // given
        EventBus eventBus = new EventBus("test");

        DeadEventListener deadEventListener = new DeadEventListener();
        eventBus.register(deadEventListener);

        // when
        eventBus.post(new OurTestEvent(200));

        assertThat(deadEventListener.isNotDelivered()).isTrue();
    }
```

So if there was no listener waiting for event which was posted, EventBus fires DeadEvent so we could do something or at least add info to log files to be aware that such situation occured.


#### Events hierarchy


Another interesting feature is that listeners can leverage existing events hierarchy. So if Listener A is waiting for events A, and event A has a subclass named B, this listener will receive both type of events: A and B. This can be presented with short example. Let's create two listeners, one listening for Number class events and second one for Integer (which in case you didn't know is a subclass of Number :) ):

``` java

public class NumberListener {

    private Number lastMessage;

    @Subscribe
    public void listen(Number integer) {
        lastMessage = integer;
    }

    public Number getLastMessage() {
        return lastMessage;
    }
}

```
``` java
public class IntegerListener {

    private Integer lastMessage;

    @Subscribe
    public void listen(Integer integer) {
        lastMessage = integer;
    }

    public Integer getLastMessage() {
        return lastMessage;
    }
}

```

And a test method showing this feauture:

``` java
    @Test
    public void shouldGetEventsFromSubclass() throws Exception {

        // given
        EventBus eventBus = new EventBus("test");
        IntegerListener integerListener = new IntegerListener();
        NumberListener numberListener = new NumberListener();
        eventBus.register(integerListener);
        eventBus.register(numberListener);

        // when
        eventBus.post(new Integer(100));

        // then
        assertThat(integerListener.getLastMessage()).isEqualTo(100);
        assertThat(numberListener.getLastMessage()).isEqualTo(100);

        //when
        eventBus.post(new Long(200L));

        // then
        // this one should has the old value as it listens only for Integers
        assertThat(integerListener.getLastMessage()).isEqualTo(100);
        assertThat(numberListener.getLastMessage()).isEqualTo(200L);
    }
```

In this method we see that first event (_new Integer(100)_) is received by both listeners, but second one (_new Long(200L)_) reaches only NumberListener as Integer one isn't created for this type of events.

This feature can be used to create more generic listeners listening for a broader range of events and more detailed ones for specific purposes.


### Summary


In this post I've presented some features of less known fragment of [Guava Library](http://code.google.com/p/guava-libraries/), an EventBus class. It's a very elegant and easy to apply solution when you need to listen for some events and publish them without creating sophisticated classes and interfaces hierarchy. And finally, this class is next good reason to add Guava to dependencies in your project :)
