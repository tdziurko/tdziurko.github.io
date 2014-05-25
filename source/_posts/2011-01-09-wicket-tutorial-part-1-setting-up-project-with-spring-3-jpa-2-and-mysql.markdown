---
author: Tomasz Dziurko
comments: true
date: 2011-01-09 18:58:58+00:00
layout: post
slug: wicket-tutorial-part-1-setting-up-project-with-spring-3-jpa-2-and-mysql
title: Wicket Tutorial, part 1 - setting up project with Spring 3, JPA 2 and MySQL
wordpress_id: 379
categories:
- How-to
- Item Directory application
- Java
- Wicket
- Wicket Tutorial
tags:
- How-to
- Java
- JPA
- skeleton
- Spring
- Tutorial
- Wicket
---

Today I start my personal project **Item Directory** which will be developed as a element of Wicket Tutorial series on this blog. Item Directory will be a web application to help you manage your collections of items (books, CDs, movies or even postage stamps).  Application is built with** Java, Wicket, JPA, Spring 3 and MySQL** as a database. It is a response for my family needs to manage some items hidden in shelves, bookcases and cabinets where we need something and we think "We have it somewhere, but I have no idea where it is located" :)


## Is this Wicket tutorial for me?


This tutorial is for Java developers who want learn about this great framework in practise not only from reading a book. Because I like to learn while creating something, this tutorial will guide you through development process. I will concentrate mainly on adding new features and developing project instead of explaining every single aspect of how Wicket works in its internals or every single line of code I created. But those parts I think are worth some explanation will be discussed in detail.

Of course if something seems unclear to you or you think I should write some longer explanation in some part of my post, please send a comment and I will try to answer to your needs as best as I can :) Moreover all code will be available to download from Mercurial repository, so you can easily download and deploy application with Jetty to test and see how it exactly works.

And last thing: as it is my first bigger tutorial published here, all comments, suggestions, etc. are very appreciated. Additionally, ideas what to do/implement next are also welcome and will be considered :)

<!-- more -->


## Beginning


This post will be dedicated to setting up project base with very simple (but working!) Wicket, Spring 3, JPA 2 and MySQL integration. For those who want Maven archetypes to start Wicket project in one command there are two places to go:

- [Wicket Cool](http://code.google.com/p/wicketcool/), project by [Paul Szulc](http://www.paulszulc.com/) offering rather complicated but very useful archetype for Wicket, HSQLDB, [Liquibase](http://www.liquibase.org/), JPA 2, Spring technology stack. I like it very much although it has some flaws. In my last Wicket commercial project our team used WicketCool as a base for development.

- [Wicket Leg Up](http://www.jweekend.com/dev/LegUp), project by [jWeekend](http://www.jweekend.com), providing archetype with many different technologies to use with Wicket.

My initial commit is mainly based on **WicketCool** because I like its Maven modularization and clean application architecture layers. But as it's too sophisticated for purposes of this tutorial and for ItemDirectory early development I removed some stuff to make things easier to understand. My project base does not contain: Liquibase integration, testing classes (yes, I did that intentionally to make project less complex, sorry TDD lovers), HSQLDB (I use MySQL instead). I also removed and modified some classes to make them easier to understand at the beginning of this tutorial. In some cases I plan to use those deleted elements in the future, but currently they are redundant.


## Project source code


Source code for this post can be downloaded from Bitbucket: [https://bitbucket.org/tdziurko/item-directory/get/a61e7cf66cc3.tar.gz](https://bitbucket.org/tdziurko/item-directory/get/a61e7cf66cc3.tar.gz)[ ](https://bitbucket.org/tdziurko/item-directory/src/a61e7cf66cc3)or pulled with Mercurial using command:

    
    hg clone https://bitbucket.org/tdziurko/item-directory -r "Base Project"
    


Latest application version is always available at [https://bitbucket.org/tdziurko/item-directory/overview](https://bitbucket.org/tdziurko/item-directory/overview) or with command:

    
    hg clone https://bitbucket.org/tdziurko/item-directory
    




## Project Overview


ItemDirectory is a Maven project consisting of three child projects: domain, service and webapplication. These three names are self-explanatory so it should be clear what will be located in each of these sub-projects. After downloading it you must first prepare MySQL (two scripts located in itemdirectory/itemdirectory.domain/src/main/resources/sql and then execute two Maven commands to run  application locally:

    
    mvn clean install
    


in itemdirectory dir and:

    
    mvn jetty:run


in itemdirectory\itemdirectory.webapplication dir.

And after a while when Maven tries to download half of the Internet, you could type [http://localhost:9090/item-directory/](http://localhost:9090/item-directory/) to see following page:

[![](/images/blog/2011/01/ItemDirectoryHello.png)](/images/blog/2011/01/ItemDirectoryHello.png)


## First step - make this technology stack work for me


Note: this part should be read with downloaded source code as in some cases I won't post complete and every code listing.

In domain project we have few elements worth noting:



	
  1. Directory src/main/resources/sql with two SQL scripts to create database and create first table.

	
  2. One interface (_IEntity_) and one abstract class (_AbstractEntity_) for entity classes to gather all repeating code in one place.

	
  3. One interface (_DAO_) and one abstract class (_AbstractDAO_) for all DAO classes.

	
  4. Simple _User_ class to represent user of our application.

	
  5. _UserDao_ and _UserDaoImpl_ classes for access to _users_ table.

	
  6. _domain-context.xml_ file with Spring general configuration (data source, transactions, enabled beans config with annotations, etc.).


In service project there are only service classes for actions connected with User class. Service layer is the place where all application logic will be located.

And now, the most important part of this post: webapplication project and its contents. This part of ItemDirectory application needs more detailed explanation.

**Web.xml, deployment descriptor:**

[xml]
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://java.sun.com/xml/ns/j2ee"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://java.sun.com/xml/ns/j2ee http://java.sun.com/xml/ns/j2ee/web-app_2_4.xsd"
	version="2.4">
	
    <display-name>Item Directory</display-name>
	
    <filter>
        <filter-name>openEntityManagerInViewFilter</filter-name>
        <filter-class>
            org.springframework.orm.jpa.support.OpenEntityManagerInViewFilter
        </filter-class>
    </filter>

    <filter-mapping>
        <filter-name>openEntityManagerInViewFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/applicationContext.xml</param-value>
    </context-param>
    
    <listener>
        <listener-class>
            org.springframework.web.context.ContextLoaderListener
        </listener-class>
    </listener>

	<filter>
		<filter-name>wicket.filter</filter-name>
		<filter-class>org.apache.wicket.protocol.http.WicketFilter</filter-class>
		<init-param>
			<param-name>applicationFactoryClassName</param-name>
			<param-value>
				org.apache.wicket.spring.SpringWebApplicationFactory
			</param-value>
		</init-param>
	</filter>

	<filter-mapping>
		<filter-name>wicket.filter</filter-name>
		<url-pattern>/*</url-pattern>
	</filter-mapping>


</web-app>
[/xml]

What do we have here?

First, some Spring-related configuration: _Open Session In View_ filter (or _Open Entity Manager in View_ like they call it for JPA), location of applicationContext.xml and context loader listener to allow our application interact with Spring. And after this there are elements indispensable in every Wicket application: declaration of Wicket filter and Wicket Application class. The only thing to add is that thanks to the _SpringWebApplicationFactory_ instance of our Application object will be created and managed by Spring.

**Wicket Application class:**

Next very important file is class _Application.java_, the heart of every Wicket application. From here we can change every aspect of both Wicket and our application configuration:

[java]
@Component(value = "wicketApplication")
public class Application extends WebApplication {

	private static final String DEFAULT_ENCODING = "UTF-8";

	@Autowired
        private ApplicationContext applicationContext;

	@Override
	protected void init() {
		super.init();
		addComponentInstantiationListener(new SpringComponentInjector(this, applicationContext, true));

		getMarkupSettings().setDefaultMarkupEncoding(DEFAULT_ENCODING);
		getRequestCycleSettings().setResponseRequestEncoding(DEFAULT_ENCODING);

		// (...)

		if (getConfigurationType().equals(WebApplication.DEPLOYMENT)) {
			getMarkupSettings().setStripWicketTags(true);
			getMarkupSettings().setStripComments(true);
			getMarkupSettings().setCompressWhitespace(true);
		}

	}

    // (...)

	@Override
	public String getConfigurationType() {
		return WebApplication.DEVELOPMENT;
	}

	@Override
	public Class getHomePage() {
		return HomePage.class;
	}

	public static Application get() {
		return (Application) WebApplication.get();
	}

}
[/java]

Interesting places in this class are:



	
  * _@Component _declaration to inform Spring about our Wicket application class which needs to be instantiated.

	
  * _@Autowired applicationContext_ to allow Spring to inject beans in Wicket components.

	
  * overridden method _init()_ to change default configuration: character encoding, Spring component to inject beans and flags to make sure that our application deployed on the production server will not contain redundant white spaces and Wicket tags in HTML code

	
  * declared home page of our application

	
  * application configured to work in development mode that will allow us to debug errors faster and easier.


**Classes for web pages:**

Every web application should have at least one web page to present some information to the user. We will start with two  classes: _BasePage_ will be a skeleton of every page in ItemDirectory application. This class in the future will contain common layout, top and left menu which in most cases will be the same for every page in our web app. In HTML file for this class there is one interesting element: _wicket-child_ a place-holder for content added by pages extending BasePage class:

[html]
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en"
	xmlns:wicket="http://wicket.apache.org/dtds.data/wicket-xhtml1.4-strict.dtd">
<head>
</head>
<body>

	<wicket:child/>

</body>
</html>
[/html]

With this abstract page our only one concrete web page HomePage.html can look like that:

[html]
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html xmlns="http://www.w3.org/1999/xhtml" xml:lang="en" lang="en"
	xmlns:wicket="http://wicket.apache.org/dtds.data/wicket-xhtml1.4-strict.dtd">
<head>
    <wicket:head>
		<title>Item Directory v0.1</title>
    </wicket:head>
</head>
<body>
	<wicket:extend>
		<h1 wicket:id="helloLabel"></h1>
		<span wicket:id="currentTime"></span>
		<br/>
		<br/>
		Number of user in database: <span wicket:id="numberOfUsers"/>
	</wicket:extend>
</body>
</html>
[/html]

Wicket while rendering this web page will omit everything placed outside _wicket-extend_ tags except some places marked with other Wicket tags (like _ wicket:head_ which allow to add some data from concrete class to web page header). Framework in render phase will place all elements between _wicket-extend_ from HomePage.html in BasePage.html file replacing _wicket:child_ tag. This works like simple inheritance: child page tells its parent what content to render in the defined place in parent's content.

And as you should already know, like in every Wicket application our _HomePage.html_ file needs accompanying Java class _HomePage.java_:

[java]
import java.util.Date;
import org.apache.wicket.markup.html.basic.Label;
import org.apache.wicket.spring.injection.annot.SpringBean;
import pl.tomaszdziurko.itemdirectory.service.users.UserService;
import pl.tomaszdziurko.itemdirectory.web.BasePage;

public class HomePage extends BasePage {

	@SpringBean
	private UserService userService;

	public HomePage() {
		initGui();
	}

	private void initGui() {
		add(new Label("helloLabel", "Wicket is saying 'Hello' to you!"));
		add(new Label("currentTime", new Date().toString()));

		add(new Label("numberOfUsers", userService.size() + ""));
	}

}
[/java]

In this class @SpringBean annotation is responsible for providing access to beans (UserService bean in this case) and method _initGui()_ contains three labels to render something to the user. The last label confirms that our access to the database through service layer, domain layer to MySQL is working properly.

And this is the end of the first part of Wicket tutorial. I hope you find it useful. Any comments appreciated! In next post we will talk about the first version of application layout and how to add new data to database.
