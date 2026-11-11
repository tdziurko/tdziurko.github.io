---
layout: post
title: The life cycle of a servlet
description: Servlet life cycle from the perspective of the container and the user
date: 2010-04-21
author: tomek
image: '/images/servlet-life-cycle-cover.jpg'
tags: [Programming, Java]
tags_color: '#5fb05f'
redirect_from:
  - /2010/04/the-life-cycle-of-servlet/
---

## Life cycle of a servlet from the perspective of the container

1. After start, the container is looking for servlet classes in the appropriate directory (for Apache Tomcat it would be `$TOMCAT_HOME/webapps`).
2. Once they are found, there are two possible ways: servlets are loaded immediately or only when someone wants to use them. By using the term “load servlet” I mean the execution of its constructor and init() method. init() is executed only once in the life cycle of a servlet. It allows you to prepare it for usage (e.g. to connect to the database, initialize any resources).

The code below presents a fragment of the web.xml file allowing to define the order in which servlets are loaded when the container starts:

```xml
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

Regarding the _load-on-startup_ element, the specification states that:

- The value of the element must be an integer
- Servlets are loaded starting from the one with number 0
- For values lower than 0 the container can load the servlet at any time, the same behaviour exists when there is no _load-on-startup_ element in the web.xml
- For servlets with the same value of the _load-on-startup_ element, the container can load them in any order

But back to our life cycle:

3. Execution of service() method for incoming requests.
4. Execution of destroy() method which allows the servlet to release its resources. This method is executed only once in the whole life cycle.
{: start="3"}

## Life cycle of a servlet from the user’s perspective

1. User enters url matching servlet class.
2. Container seeing that the address refers to the servlet, creates two objects implementing interfaces _HttpServletRequest_ and _HttpServletResponse_, then finds appropriate servlet (defined in web.xml), **creates a new thread** and calls service() method passing request and response objects as arguments.
3. Method service() calls doXXX() method on the basis of the type of HTTP method. Most often it’s doGet() or doPost(). Same arguments (req, resp) are passed to this method.
4. Called method generates response and writes it to the received _HttpServletResponse_ object.
5. service() method ends, response is sent to the user, thread dies (or returns to the pool). Cycle ends.

