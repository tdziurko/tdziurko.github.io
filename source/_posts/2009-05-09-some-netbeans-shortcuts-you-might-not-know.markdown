---
author: Tomasz Dziurko
comments: true
date: 2009-05-09 18:26:14+00:00
layout: post
slug: some-netbeans-shortcuts-you-might-not-know
title: Some NetBeans shortcuts you might not know
wordpress_id: 103
categories:
- How-to
tags:
- How-to
- IDE
- NetBeans
- productivity
- shortcuts
---

Every developer has favourite Java framework, library or  favourite browser, the more advanced even have favourite design pattern :) Each  one of us also have his beloved IDE. The most popular are NetBeans and Eclipse, both with many admirers and  enemies who is always ready to argue that one IDE is great, and the other is passe. From time to time, unaware novice developer asks in one or another discussion group "Which IDE choose : Eclipse or NetBeans?" causing  endless debate very similar to that in which some people try to  prove the superiority of Christmas over Easter, while the other opts for  the opposite proposition.

﻿<!-- more -->My journey with Java began five years ago and that time the reason I chose IDE was very casual. I just asked my room-mates which one they were using. I thought  that when I will be learning Java language it will be easier for them to explain something to me if  I have the same IDE  as they use :)

As you can guess from the title of the post, I chose NetBeans, and I am still using it. At my new job everyone uses Eclipse. After initial reluctance to this IDE  I got used to it and now I am kind of multi-IDE guy :) Today I will write about a bit of keyboard shortcuts in NetBeans. Some  of them are probably known by you, but I hope that a few "pearls" of  this post will join the set of your most frequently used shortcuts.


### Fast System.out.println("");


One of the most often used shortcuts. In the project, when you don't want to add Log4J logging, debugging is often done by printing messages to the console. System.out.println("I am in methodName"); is a common thing in small programs developed in Java. And instead of  painful typing the whole command by hand, we can type just **sout** and press** Tab**. NetBeans will automagically generate command for us and, as a bonus, will place cursor between quotation marks so we could start typing our message straight away.

Moreover, there is another variant of this shortcut allowing to easily create command like System.out.println("value = " + value); Suppose we have following code:

``` java
counter = countAllActiveAccounts();
```

Then we place cursor in the line below and type **soutv** and press **<Tab>**. What we get is:

``` java
counter = countAllActiveAccounts();
System.out.println("counter = " + + counter);
```

NetBeans found the closest variable and generated proper println method. And, last extra add-on here. if we start typing something just after this command we can end with something like this (I used sequence **soutv + <Tab> + anotherVariable**):

``` java
System.out.println("anotherVariable = " + anotherVariable)
```


### Fast commenting and uncommenting many lines of code


If we want comment many lines of code we can use /* ... */ but NetBeans gives us a better and faster way to do it with **<Ctrl> + /** shortcut:

Method before we mark it and use the shortcut:


->![](/images/blog/2009/05/uncommented1.png)<-


Method after:


->![](/images/blog/2009/05/commented1.png)<-





And of course using the same shortcut second time uncomment selected part of code.

East navigating to proper class, interface

In the large project with many packages, classes and interfaces looking for the one we need might be cumbersone. One of the solutions is **<Ctrl> + o** which opens window when you can start typing name of the type you are looking for. The more you type, the more narrow results will be:

->![](/images/blog/2009/05/ctrlO.png)<-


### Go to the source with one click


When you want to examine source code of any class there is one, very easy way to do it, just press **<ctrl> + LMB **(Left Mouse Button) on class or interface name. And if you have source code connected to any jar you use in NetBeans, you can go even there and see how it works from the inside.


### Fast renaming


Sometimes you need to rename variable or class in you project. To make it easy you just have to place cursor on variable and press **<Ctrl> + r** and when you change name in once place, NetBeans will other occurrences too. Same applies to renaming classes, interfaces etc.


### Move left, move right, move up, move down - move your code easily


If you want to move whole line or lines of code you can use:



	
  * **<Ctrl>+<Shift>+UP/DOWN** to move code up/down

	
  * **<Ctrl>+<Shift>+LEFT/RIGHT **to move code left/right for one tabulation. You can achieve the same result with using **<Tab>** (one tab to the right) or **<Ctrl>+<Tab> **(one tab to the left)




### Other useful shortcuts





	
  * **<Alt>+<Insert>** - generate getters/stters, constructors and so on

	
  * **<Ctrl> + <Shift> + UP/DOWN** - clone selected code to the line above/below

	
  * **<Shift> + V + <Ctrl>** - paste text and format it in one step

	
  * **<Ctrl> + e** - delete current line

	
  * **<Ctrl> + <Tab>** - press it for a moment to change view for the next opened file. Press it for longer to choose which one file you want to view.


Uff.. enough for today :) I suggest trying these one in practice just to memorize them so you can use them in your everyday work.
