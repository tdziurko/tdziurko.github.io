---
author: Tomasz Dziurko
comments: true
date: 2011-07-15 22:05:09+00:00
layout: post
slug: wicket-tutorial-part-9-%e2%80%93-storing-loading-user-preferences-cookies
title: Wicket Tutorial, part 9 – storing and loading user preferences from cookies
wordpress_id: 797
categories:
- How-to
- Item Directory application
- Java
- Wicket
- Wicket Tutorial
tags:
- cookies
- Tutorial
---

In the previous post we learnt a few tricks regarding internationalization features in Wicket. Today we will go further and will let application to remember some user preferences using HTTP Cookies. Storing some basic data in cookies is an easy and popular way of letting user customize look and feel of web application he uses very often. Some portals use cookies to render modules in order chosen by user so that most interesting information are in the top.

In our ItemDirectory web application we don't have (yet!) any sophisticated mechanism which could rely on cookies so we will show how it can be used to store and load language selected by the user.

<!-- more -->


### Storing information in cookies


So first thing we need to do is store which language user wants to use. So we must modify changeUserLocaleTo() method in LanguagePanel class:

``` java
public class LanguagePanel extends Panel {

  // (...)

  private void changeUserLocaleTo(String localeString) {
    getSession().setLocale(new Locale(localeString));

    Cookie languageCookie = new Cookie(Application.LANGUAGE_COOKIE_NAME, localeString);
    languageCookie.setMaxAge(Application.LANGUAGE_COOKIE_AGE);
    ((WebResponse)getResponse()).addCookie(languageCookie);
  }
}
```

This mechanism is rather easy to understand. Besides setting session locale we also create and put language cookie with name and age defined as static fields in our Application class. We just have to remember to add our new or modified cookie to response so it will be send back to the browser.
After this we can run jetty with deployed application and check that after clicking there is a cookie with information about selected language.


### Loading information from cookies


Now we have some information in cookie waiting for our application to use it. The best moment to utilize such information is while creating new session so application could load preferences from cookie and store it in user session object. So lets play with method in WebApplication class which creates new user session:

``` java
public class Application extends WebApplication {

  // (...)

  @Override
  public Session newSession(Request request, Response response) {

    Session session = super.newSession(request, response);
    session = trySetLanguageFromCookie(session, request, response);

    return session;
  }
}
```

So far so good, but the most important things happen in trySetLanguageFromCookie() method

``` java
private Session trySetLanguageFromCookie(Session session, Request request, Response response) {

  Cookie[] cookies = ((WebRequest) request).getCookies();

  if (cookies == null || cookies.length == 0) {
    return session;
  }

  for (Cookie cookie : cookies) {
    if (LANGUAGE_COOKIE_NAME.equals(cookie.getName())) {  // (1)
      session.setLocale(new Locale(cookie.getValue()));   // (2)

      cookie.setMaxAge(LANGUAGE_COOKIE_AGE);
      ((WebResponse)response).addCookie(cookie);          // (4)
      break;
    }
  }

  return session;
}
```

In this method we first try to find **(1)** cookie named LANGUAGE_COOKIE_NAME and if we find it, we modify **(2)** session locale according to data loaded from the cookie. And another thing we could do is refresh **(4)** just loaded cookie so if user won't click flag for a long time (as he doesn't have to because we automatically set proper locale basing on cookie), he won't lose his preference.

Now, everything should work fine. User can choose language when he enters our application for a first time and later we load this information from cookie, set proper locale and refresh cookie so it won't expire.
