---
layout: post
title: "One and the Only One Reason to Customize IntelliJ IDEA Memory Settings"
description: Benchmark of four IntelliJ IDEA memory settings on a big monolith and microservices
date: 2015-11-23
author: tomek
image: '/images/intellij-memory-settings.jpg'
tags: [Programming, Java]
tags_color: '#5fb05f'
redirect_from:
  - /2015/11/1-and-the-only-one-to-customize-intellij-idea-memory-settings/
---

_Don’t be a Scrooge and give your IDE some more memory_

Yesterday we had a discussion about customizing memory settings in IntelliJ IDEA and it appeared that some people do not do that, some people (like me) use some simple changeset and some developers craft a fancy, sophisticated setup that satisfies their needs. While working for a current project I had to deal with several small microservices projects and one older project, a core of the client’s business which is quite big. And after applying some changes I have noticed a really significant improvement when it comes to IDE speed and responsiveness. But at that time I didn’t measure what exactly changed, it was only a subjective observation that _“IDEA is now faster”_.

But during that discussion one of the developers working for the same client (thank you Yuri) sent me his settings and I was really overwhelmed by their level of complexity. I was happy with my own setup, but I was also curious how those completely different settings compare to each other and also how they compare to the defaults provided by JetBrains. So after the discussion with Yuri I knew that this is a good material for a blog post.

## Goal

My plan is to compare how different IDEA memory settings perform in a scenario close to my daily usage (load a big project, load two or three microservices, refresh the big project after hypothetical _git pull_) and then choose the most optimal settings when it comes to memory consumption and speed improvement.

## Test machine and projects

**Laptop**: MacBook Pro Retina, 2,3GHz Intel Core i7, 16GB 1600Mhz DDR3, SSD Disc, OS X Yosemite

**Projects**

- Big Project – monolith, 700 000 lines of code (Java 8 and Groovy), 303 Gradle modules
- Two microservices – small projects with 10 000 – 20 000 lines of code (Java 8 and Groovy), each with one Gradle module

## Test scenario

1. Close all projects in Idea
2. Put settings under test to idea.vmoptions file
3. Reset my laptop
4. After a start close all unrelated programs (communicators, etc.)
5. Open Idea (measure time)
6. Open Big Project (measure time)
7. Check jstat -gcutil
8. Open two more projects: Microservice One and Microservice Two (measure times)
9. Check jstat -gcutil
10. Go back to Big Project and click “Refresh Gradle project” button (measure time)
11. Check jstat -gcutil

## What is jstat -gcutil

jstat is a tool available in your JDK to monitor JVM and Garbage Collector statistics. It has many different options to collect various data ([full documentation is here](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/jstat.html)) but we are interested in only one: **-gcutil**:

```text
-gcutil - Summary of garbage collection statistics.

S0: Survivor space 0 utilization as a percentage of the space's current capacity.
S1: Survivor space 1 utilization as a percentage of the space's current capacity.
E: Eden space utilization as a percentage of the space's current capacity.
O: Old space utilization as a percentage of the space's current capacity.
M: Metaspace utilization as a percentage of the space's current capacity.
CCS: Compressed class space utilization as a percentage.
YGC: Number of young generation GC events.
YGCT: Young generation garbage collection time.
FGC: Number of full GC events.
FGCT: Full garbage collection time.
GCT: Total garbage collection time.
```

Example output from this command looks like this:

```text
S0     S1    E     O     M    CCS  YGC YGCT FGC  FGCT   GCT
89.70 0.00 81.26 74.27 95.68 91.76 40 2.444 14  0.715  3.159
```

In this post the most important parameters are number of GC events (**YGC** and **FGC**) and collection times (**YGCT** and **FGCT**).

## Tested settings

I have tested four different settings, to make reading easier each of them was given a name.

### Default (grey)

These are built-in settings provided by JetBrains, clean IDEA 15 is using them:

```text
-Xms128m
-Xmx750m
-XX:MaxPermSize=350m
-XX:ReservedCodeCacheSize=240m
-XX:+UseCompressedOops
```

### Big (red)

4096MB for Xmx and 1024MB for _ReservedCodeCacheSize_, that’s quite a lot of memory.

```text
-Xms1024m
-Xmx4096m
-XX:ReservedCodeCacheSize=1024m
-XX:+UseCompressedOops
```

### Balanced (blue)

2GB for Xmx and 2GB for Xms, more balanced approach to the memory consumption

```text
-Xms2g
-Xmx2g
-XX:ReservedCodeCacheSize=1024m
-XX:+UseCompressedOops
```

### Sophisticated (orange)

2GB for Xmx and 2GB for Xms as above, but different Garbage Collector is specified and many different flags for GC and memory management. I have received these settings from Yuri.

```text
-server
-Xms2g
-Xmx2g
-XX:NewRatio=3
-Xss16m
-XX:+UseConcMarkSweepGC
-XX:+CMSParallelRemarkEnabled
-XX:ConcGCThreads=4
-XX:ReservedCodeCacheSize=240m
-XX:+AlwaysPreTouch
-XX:+TieredCompilation
-XX:+UseCompressedOops
-XX:SoftRefLRUPolicyMSPerMB=50
-Dsun.io.useCanonCaches=false
-Djava.net.preferIPv4Stack=true
-Djsse.enableSNIExtension=false
-ea
```

So these are our test setups. To perform our test scenarios we have to create a file _idea.vmoptions_ under _~/Library/Preferences/IntelliJIdea15/_ (it is Mac OS specific, to see how to change these settings for your OS, check [this article](https://www.jetbrains.com/idea/help/increasing-memory-heap.html)).

Now it is time to perform our test scenarios and compare the results.

## Results

{: .tip }
Note: the charts with the results were published from Google Sheets and are no longer available, so unfortunately they could not be migrated together with this post. The descriptions below still summarize the outcome of each test.

### Idea startup time

As you can see startup time does not depend on memory settings. Idea startup time is about 10 seconds for all test scenarios no matter how much memory we allocate. And this is not a surprise as those settings should not affect application behaviour at such an early stage.

### Time to load The Big Project

Ok, so now it is time to load our Monolith and its 700k lines of code.

Finally there are some differences. Default settings perform almost three times worse than the rest. Apparently such a large codebase needs more memory 🙂 And if we execute

```bash
jstat -gcutil <IDEA_PID>
```

we will notice that Garbage Collector was really, really busy with default settings when we compare it to other setups.

Not only is the total time spent by GC on freeing memory dramatically higher (about 50 times higher) but also average execution time of Full GC is much, much longer. Long periods spent on Full GCs are the main cause of low responsiveness of our IDE.

### Opening two microservices in IDEA

Ok, so we have our Monolith loaded but we need to add some code to two smaller microservices. So let’s open them in IDEA and compare total times.

In this test scenario we see that differences are still visible and **Sophisticated** wins here, **Default** is far behind the rest.

### jstat -gcutil again

After loading two microservices we can check how the Garbage Collector is performing with three opened projects. We can see that three custom settings look almost the same and results of default settings are very, very poor.

### Final stage: reload the Monolith

So we have coded for a while and now we need to fetch the latest version of The Big Project from repository and then refresh Gradle Project so IDEA could see all new classes, etc.

**Important note**: Bar for Default setup is so high because IDEA crashed during refresh and I could not measure the actual time. Simply, the assigned memory was not sufficient to perform this operation.

But looking at the best three, we see that Big settings refreshed project in the shortest time so the largest assigned memory helps a bit here.

### jstat -gcutil for the last time

Because IDEA was not able to refresh the project with **Default** settings, they are not included in this measurement.

In these last charts we see that differences between total times are quite small, but single Full GC is the fastest with **Big** settings applied. Again, it looks like very large Xmx helps a bit more when it comes to responsiveness.

## Summary

In this short experiment I tried to test how much we could gain by customising memory settings of IntelliJ IDEA and it looks like even some simple tweaks can drastically improve performance of our IDE and speed up our work. Of course the more memory you assign, the better results you will see, but we are using IDE next to many different applications which also consume memory so our goal should be to find a balance between performance gain and memory consumption. I think that in most cases setting **Xmx** to a value between 2g and 3g is the best approach. If you have some more time you can play with jstat and [jvisualvm](http://docs.oracle.com/javase/6/docs/technotes/tools/share/jvisualvm.html) to check how changing different VM flags affects performance and memory footprint.

### And you?

And what about you? What is your idea.vmoptions setup? Do you have any other ways to improve IntelliJ IDEA performance?
