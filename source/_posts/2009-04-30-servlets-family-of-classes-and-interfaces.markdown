---
author: Tomasz Dziurko
comments: true
date: 2009-04-30 07:03:40+00:00
layout: post
slug: servlets-family-of-classes-and-interfaces
title: Servlets –  family of classes and interfaces
wordpress_id: 94
categories:
- Certificates
- Java
tags:
- Certificates
- Java
- SCWCD
- Servlets
---

Today I am going to present the most important methods, classes and interfaces in the Servlet API.


### public interface Servlet


Main interface from which all starts. It defines key methods for the life cycle of servlet:

[java]

public void init(ServletConfig config) throws ServletException;
public ServletConfig getServletConfig();
public void service(ServletRequest req, ServletResponse res) throws ServletException, java.io.IOException;

public void destroy();

[/java]

and one additional method

[java]

public String getServletInfo();

[/java]

which should return information about servlet (version, author, resposibility, etc.)<!-- more -->


### **public abstract class GenericServlet implements Servlet, ServletConfig**


Abstract  class implementing all Servlet interface methods, except method service (), which is implemented by a class one step  lower in the hierarchy: HttpServlet. Additionally GenericServlet also provides methods from ServletConfig interface allowing developer to read init parameters of servlet

[java]

public String getInitParameter(java.lang.String name) { }
public Enumeration getInitParameterNames() { }

[/java]

These  parameters are defined in the web.xml file by using the elements <init-param> nested in the declaration of a servlet:

[xml]

<web-app>
  <servlet>
    <servlet-name>FirstToLoadServlet</servlet-name>
    <servlet-class>pl.tdziurko.scwcd.servlets.FirstToLoadServlet</servlet-class>
    <init-param>
      <param-name>initParameter</param-name>
      <param-value>initValue</param-value>
    </init-param>
    <init-param>
      <param-name>anotherInitParameter</param-name>
      <param-value>anotherInitValue</param-value>
    </init-param>
  </servlet>
  <servlet-mapping>
    <servlet-name>FirstToLoadServlet</servlet-name>
    <url-pattern>/FirstToLoadServlet</url-pattern>
  </servlet-mapping>
</web-app>

[/xml]

Another method of the interface ServletConfig

[java]

public ServletContext getServletContext () {}

[/java]

returns the context in which servlet is running. From this context we can get information about server version and many other things which I don't cover in this episode of "Preparing to SCWCD" series :)

The final method in class GenericServlet worth mentioning is

[java]public void init () throws ServletException {}

[/java]

This method is added for the developers convenience. Rather  than overloading the init (ServletConfig config) and remember to call  super.init (config) , we can put our init logic in servlet initialization  method init (), which is called by the init (ServletConfig config).


### public abstract class HttpServlet extends GenericServlet


It's the class, which will be extended by all our servlets. It has methods for handling HTTP requests:

[java]

protected void doGet (HttpServletRequest req, HttpServletResponse resp) throws ServletException, java.io.IOException {}

[/java]

and its brothers working with other HTTP protocol requests: doPost, doHead, doOptions, doPut, doDelete and doTrace.

The two most important methods in this class are:

[java]

public void service (ServletRequest req, ServletResponse res) throws ServletException, java.io.IOException {}
protected void service (HttpServletRequest req, HttpServletResponse resp) {}

[/java]

The first one (public one) is called by the container and redirects call to second one (protected), which, depending on the type of request calls one of doXXX() method.
