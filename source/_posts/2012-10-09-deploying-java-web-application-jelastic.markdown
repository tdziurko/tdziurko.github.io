---
author: Tomasz Dziurko
comments: true
date: 2012-10-09 19:44:34+00:00
layout: post
slug: deploying-java-web-application-jelastic
title: Deploying Java Web Application on Jelastic
wordpress_id: 1647
categories:
- Item Directory application
- Java
- Wicket
- Wicket Tutorial
tags:
- Java
- Jelastic
- SaaS
- Wicket
---

Some time ago having your own full-fledged hosting with Java, Tomcat and any database wasn't cheap, but luckily we live in a very interesting times and now there are many virtual hosting services in the Cloud providing an easy and affordable way to deploy your application and make it available to users.

And what is the best in such solutions, they grow with your application or startup. Services are free or almost free when your web application doesn't need much memory and processors but when your web-app starts to gather more users' attention and you have more and more concurrent sessions and visits each day, you could easily enhance your setup to match your needs. Everything is a matter of a few clicks, simple and fast.

To test how this deployment process look like, I've decided to put my [Wicket tutorial](<a href="/2011/03/wicket-tutorial-series-building-web-application-scratch/) web application online. It consist of three pages and two tables in database so I think it shouldn't be a too complicated process to deploy it online. And as a Java hosting service I've chosen [Jelastic](http://jelastic.com), recent winner of [2012 Duke's Choice Awards](http://www.java.net/dukeschoice) announced during JavaOne conference this year. Duke's Choice Award is something like a Oscar in the Java world, so I thought that Jelastic might be a good choice to start with.<!-- more -->


### First steps


So after creating an account and choosing a hosting in Germany (which is the closest one to Poland where I live) I am redirected to my personal administration panel where in a straightforward step-by-step wizard I am requested to choose container /server (Tomcat 6/7,  Jetty amd Glassfish are available) and then my subdomain name. After that my server is deployed and after a moment I could check that [http://tomaszdziurko.jelastic.dogado.eu/](http://tomaszdziurko.jelastic.dogado.eu/) is live and running. User is also encouraged to try and deploy pre-defined Hello World application. The whole process is extremely easy, just one click and in a moment we could check that we've just deployed our first war on Jelastic:


->![](/images/blog/2012/10/jelastic-demo.jpeg)<-





### Configuring more real application environment


So far everything was easy, but we didn't touch any real scenario so now it's high time to start deploying our example Wicket application. First step is to upload application war to the Jelastic. Then we have to enable and configure our database with three tables. To do that we choose "Environment topology" next to our only server and here we could turn on/off many interesting features: load balancers, caching solutions, multiple containers, SSL-s and what we currently need SQL and/or NoSQL database:


->![](/images/blog/2012/10/dbConfig.jpeg)<-




So let's enable MySQL 5.5 and wait some time until it is configured and started on our virtual server. When everything is ready we will receive an email with login and password to our MySQL database instance. Now we could login to myPhpAdmin web interface and create user and MySQL tables required by our Wicket application. Because database url is no longer localhost:3306 we have to modify properties file in our project, rebuild it and upload war again. After that we are ready to deploy application on Jelastic. Simply, we click "Deploy" icon and then choose context name where application will be accessible. After that we must wait for "Success" message from deployment process and now we could check if [http://tomaszdziurko.jelastic.dogado.eu/item-directory](http://tomaszdziurko.jelastic.dogado.eu/item-directory) is working as expected.





### More and more options to choose and play with


This basic scenario has shown you how simple Java web application with database connecticity could be deployed on [Jelastic](http://jelastic.com). Of course in any business you will need more sophisticated stuff, but there aren't many things that you will not find available in this service. Most interesting stuff what you can do is:



	
  * link your web-app to private Git/SVN repository and deploy it directly from Jelastic admin panel

	
  * choose from multiple SQL and NoSQL databases

	
  * launch your Virtual Dedicated Server (VDS) with root privileges and full control there

	
  * use predefined load balancing and session replication configuration

	
  * add new or duplicate server or database in a few clicks

	
  * enable SSL

	
  * use latest Java 7 in your applications




### There are some cons too


Of course there are some drawbacks present. From what I've noticed there are four things I find useful but missing on Jelastic:



	
  * <del>ability to set a few environment variables without buying whole VDS. I wanted to use these variables to deploy open source project directly from Github repository but with database login/password stored not in the publicly available source code but in these variables that could be loaded later in application and used to connect to database.</del> Update: looks like it is possible. See comment below by Marina Sprawa from Jelastic

	
  * premium account type with pricing that is more suitable for people that only want to play with Jelastic and their pet projects longer than for 2 weeks using trial period

	
  * <del>ability to configure continuous deployment environment that auto deploys latest version each time when new commit appears in the repository</del> Update: it is also possible, see comment by Marina below

	
  * JBoss 6 or 7 as an application server to choose from. I am not a big fan of Glassfish so for Java EE application I would prefer to use JBoss product.




### Summary


So to sum up, service looks very interesting and if you consider using external service provider to deploy, scale and tune your production environment, Jelastic is one of the best choices currently available. Its options to quickly and easily configure load balancing and other parameters and hardware that is required when your application suddenly becomes very popular are a life-saver. Even for smaller projects there are many stuff that you will appreciate and quickly get accustomed to, so Jelastic looks as a good place to launching your startup application online.
