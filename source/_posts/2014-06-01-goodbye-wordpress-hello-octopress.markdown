---
layout: post
title: "Goodbye Wordpress. Hello Octopress"
date: 2014-06-01 15:15:09 +0200
comments: true
keywords: octopress, wordpress, migration, reasons, gotchas, issues, problems
categories: 
---

Developers are like women. From time to time they need new framework, language or sometimes, even new blogging platform.

<!-- more -->

For quite long time I've got this idea to change something on my blog. Old version was heavy, took some time to load and 
most important, template was _noisy_ and didn't look so well.

Initially, I didn't plan to migrate whole blog. I just wanted new, minimalistic theme for Wordpress, but I couldn't find anything
suitable, even among paid (premium) themes. There were some candidates but we, programmers, need one, essential feature: 
main content column should be wide and allow to present code snippets in a pretty way.

### Research

I've lost hope that I will finally find my ideal WP skin and some day at work discussion about blogging, blogging platforms emerged.
There were some sites mentioned that I haven't heard about before: [Ghost](http://ghost.org) or [Medium](http://medium.com).
Some people also started praising [Octopress](http://octopress.org). This discussion showed me that there are some interesting 
alternatives to Wordpress and encouraged me to start digging deeper. I've spent two evenings on research, even migrated my blog to 
[Ghost](http://ghost.org), but paid service and non-trivial self-hosting weren't something I was very excited of.

Then I started looking at Octopress. Firstly I tried to find theme that was minimal and suitable for blogging developer. Surprisingly I was able 
to find more than one in a very short time. And this with some more advantages of Octopress convinced me to start my migration process.
  
#### Octopress pros:

* many themes well-suited for blogging developers
* free hosting on Github
* blazingly fast, static pages
* I can write posts in my favourite IDE

{% img center /images/blog/2014/06/octopress-logo.png %}

### Migration

As I am not the very first person that made a decision to switch from Wordpress to Octopress, migration was simple and clearly explain. 

There are some good posts that helped me along the way:

* [Everything is Gray](http://everythingisgray.com/2013/06/15/on-migrating-from-wordpress-to-octopress)

* [Jason McCreary](http://jason.pureconcepts.net/2013/01/migrating-wordpress-octopress/)

* [Decodize](http://decodize.com/html/moving-from-wordpress-to-octopress/)
 
#### Minor issues

As usual, whole process is seamless but... with some small exceptions: 

* image displayed on blog posts after conversion were too big. I guess it has something with WP auto sizing feature which 
wasn't included in exported xml data. I had to manually resize some images so they could fit into the blog.

* missing images horizontal alignment. I had to manually add ->(image)<- to every image.
 
* code snippets had to be formatted from the scratch as Syntax Highlighter placeholders were different than those in Octopress
 
* default RSS url <root>/atom.xml is different than default in Wordpress: <root>/feed so I need to apply some tricks to 
make rss look the same after migration. More details about what needs to be done in the post from [Luosky](http://luosky.com/2012/07/24/create-custom-rss-feed-for-octopress/).

* keywords for blog posts weren't migrated to Octopress. I had to add them manually.

* no SEO at all. Wordpress has pretty good support to almost any SEO feature you can think about. In Octopress this requires some changes and additional work. 
Here are some good posts about SEO in Octopress:

    * [SEO for Octopress](http://xit0.org/2013/05/seo-for-octopress-websites/)
   
    * [Octopress SEO](http://learnaholic.me/2012/10/15/octopress-seo-and-disabling-the-blog-route/)
    
### Summary

So that was relation of my short journey from Wordpress to Octopress. At the end of it, I must say I am happy with the
results: page looks better, loads faster and programming part of me has a new toy to play with :) 

However, my website is still in early alpha so if you spot any issues or have any feedback what is missing or what could be 
done better, please let me know in the comments. Thanks in advance!
    
  
