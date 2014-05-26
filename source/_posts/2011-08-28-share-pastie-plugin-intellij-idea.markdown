---
author: Tomasz Dziurko
comments: true
date: 2011-08-28 19:51:16+00:00
layout: post
slug: share-pastie-plugin-intellij-idea
title: Share with Pastie - my first plugin for IntelliJ IDEA
wordpress_id: 846
categories:
- Java
tags:
- IDE
- IntelliJ IDEA
- pastie
- plugin
- sharing code
---

When I started working for a new companyy I also changed my IDE so since June I am using IntelliJ IDEA. And I can undoubtedly say that it is worth every penny or Polish Zloty to be more precise :) It is more stable than Eclipse and it has many well supported extensions which are working out of the box without any, really ANY issues.


But this is not a post about greatness of JetBrains product. Since we all are working remotely (max distance between developers is about 650km) we use plenty of tools which allow us to collaborate, communicate and share ideas through the Internet. Skype, GoogleDocs, [Planning Poker](http://planningpoker.com/), [tinyPM](http://tinypm.com), company wiki are all among them. But one thing we are doing a few times each day is sharing code fragments online. When I have problem, I post code so we can talk about it, when I have an idea how to solve something, I post code to other team members and they can say whether my solution sucks or rocks. Mostly we share code using [Pastie.org](http://pastie.org) service.




->![](/images/blog/2011/08/pastie_home.png)<-




<!-- more -->




The problem




So the typical workflow looks like that:




1. Select interesting code fragment inside Intellij IDEA (two clicks)




2. Copy it (one click)




3. Enter pastie.org (two clicks: change active window to browser, open new tab with bookmarked url address)




4. Paste code fragment there (one click)




5. Choose correct language for proper syntax highlighting (two clicks and mouse scroll to find proper language)




6. Submit a form (one click)




7. Copy link from browser address bar so I can send it to other developers (two clicks)




So seven steps every time I want to share code with someone. Seven steps,  eleven clicks and some mouse movement too every time... and it made me think about simplyfing this repeatable tasks.





### The solution


As we are all using IntelliJ IDEA and most of the code we want to share is viewed using IDEA I thought about writing a plugin which could automate sharing code fragments with Pastie.org. And after several hours I was able to release 0.1 version. We started using it and some new features I hadn't thought before were requested, also some bugs appeared too :) So I spent some more time on adding new things and fixing bugs and currently [Share with Pastie](http://plugins.intellij.net/plugin/?idea&id=6267) version 0.4 is available. And now workflow looks like that:

1. Select interesting code fragment inside Intellij IDEA (two clicks)

2. Press Ctrl+Shift+P or right mouse click and choose "Share with Pastie.org" (one click)


->![](/images/blog/2011/08/plugin_1.png)<-




3. Plugins sends selected fragment to Pastie.org. Language for syntax highlighting is chosen using file extension from file on which "Share with Pastie" was called.




4. Link to your **private** (available only through the direct link) pastie is stored in your clipboard and you are informed about this fact by green balloon.




->![](/images/blog/2011/08/plugin_2.png)<-




5. You can paste link to your teammates thousands of kilometers away from you.




So only three clicks and you have link to your shared code fragment in your clipboard. Simple and fast. No more clicking, scrolling and manual copying address from the browser.





### Extras


There are some additional features which were not planned at the begginning but after we started using plugin in our everyday work, it appeared that these add-ons would be nice and useful.

**Sharing code from console**

While developing application you might need to copy and share something not only from the opened file but from console output (or log) your application produced. So since version 0.4 plugin allows you to share code directly from the console view:


->![](/images/blog/2011/08/plugin_31.png)<-




**Pasties history**




Sometimes you might share something with plugin and then, without thinking, copy something else to the clipboard. And then what? Your link to last code shared with Pastie is lost? It would be but [Tomek Szymański](http://szimano.org/) suggested an interesting feature that older links could be stored somewhere and user should be able to access them later. So I implemented a history window with recent pasties and links to them. It is available as separate tool window at the bottom of IDEA:




->![](/images/blog/2011/08/plugin_4.png)<-




Pasties are listed there starting from the newest and each item has its link which can be copied to clipboard too.





### Summary


"Share with Pastie" can be downloaded from central Intellij IDEA plugin repository - [link](http://plugins.intellij.net/plugin/?idea&id=6267).

Source code is available on GitHub - [link](https://github.com/softwaremill/idea-pastie-plugin)


If you want to receive notifications when new version is released you can [follow me](http://twitter.com/#!/TomaszDziurko) on Twitter, check plugin home page or its repository. If you have any suggestions, improvements, etc. you can always contact me via e-mail, Twitter or GitHub private message.




I will try to write some more about my general thoughts and experiences about developing plugin for IntelliJ IDEA soon in a separate post.
