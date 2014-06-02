---
author: Tomasz Dziurko
comments: true
date: 2010-04-21 14:55:08+00:00
layout: post
slug: the-life-cycle-of-servlet
title: The life cycle of servlet
wordpress_id: 56
keywords: java, servlet, lifecycle
categories:
- Certificates
- Java
tags:
- Certificates
- Java
- SCWCD
- Servlets
---

**Life cycle of servlet from the perspective of the container**

1. After  start, the container is looking for servlet classes in the appropriate directory (for Apache Tomcat  would be $ TOMCAT_HOME / webapps) .

2. Once they are found, there are two possible ways: servlets are loaded immediately or only when someone wants to use them. By using term "load servlet" I mean the execution of its constructor and init () method. init () is executed only once in the life cycle of servlet . It allows you to prepare it for usage (eg to connect to the database, initialize any resources).

Code below presents fragment of web.xml file allowing to define order in which servlets are loaded when the containter starts:

``` xml
<web-app>
    ...
    <servlet>
        <servlet-name>FirstToLoadServlet</servlet-name>
        <servlet-class>pl.tdziurko.scwcd.servlets.FirstToLoadServlet</servlet-class>
        <load-on-startup>0</load-on-startup>
    </servlet>
    <servlet>
        <servlet-name>SecondToLoadServlet</servlet-name>
        <servlet-class>pl.tdziurko.scwcd.servlets.SecondToLoadServlet</servlet-class>
        <load-on-startup>1</load-on-startup>
    </servlet>
    ...
</web-app>
```

<!-- more -->


Regarding the _load-on-startup_ element specification states that:



	
  * The value of the element must be an integer

	
  * Servlets are loaded starting from the one with number 0

	
  * For values lower than 0 container can load servlet at any time, same behaviour exists where there is no _load-on-startup_ element in the web.xml

	
  * For servlets with the same value of _load-on-startup _element, container can load them in any order


But back to our life cycle:

3. Execution of service() method for incoming requests.

4. Execution of destroy() method which allows servlet to release its resources. This method is executed only once in whole life cycle.

**Life cycle of servlet from the user's perspective**

1. User enters url matching servlet class.

2. Container seeing that the address refers to the servlet, creates to object implementing interfaces _HttpServletRequest_ and _HttpServletResponse_, then finds appropriate servlet (defined in web.xml), **creates new thread** and calls service() method passing request and response objects as arguments.

3. Method service() calls doXXX() method on the basis of type of HTTP method. Most often it's doGet() or doPost(). Same arguments (req, resp) are passed to this method.

4. Called method generates response and writes it to received _HttpServletResponse _object.

5. service() method ends, response is sent to the user, thread dies (or returns to the pool). Cycle ends.
