---
layout: post
title: Wicket Tutorial, part 9 – storing and loading user preferences from cookies
description: Remembering the language selected by the user with HTTP cookies in our Wicket tutorial application
date: 2011-07-16
author: tomek
image: '/images/wicket-tutorial-9-cover.jpg'
tags: [Programming, Java]
tags_color: '#5fb05f'
redirect_from:
  - /2011/07/wicket-tutorial-part-9-storing-loading-user-preferences-cookies/
---

In the [previous post](/blog/wicket-tutorial-part-8-adding-internationalization-support) we learnt a few tricks regarding internationalization features in Wicket. Today we will go further and will let the application remember some user preferences using HTTP Cookies. Storing some basic data in cookies is an easy and popular way of letting the user customize the look and feel of a web application he uses very often. Some portals use cookies to render modules in the order chosen by the user so that the most interesting information is at the top.

In our ItemDirectory web application we don’t have (yet!) any sophisticated mechanism which could rely on cookies so we will show how it can be used to store and load the language selected by the user.

## Storing information in cookies

So the first thing we need to do is to store which language the user wants to use. So we must modify the changeUserLocaleTo() method in LanguagePanel class:

```java
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

This mechanism is rather easy to understand. Besides setting the session locale we also create and put language cookie with name and age defined as static fields in our Application class. We just have to remember to add our new or modified cookie to the response so it will be sent back to the browser.

After this we can run jetty with the deployed application and check that after clicking there is a cookie with information about selected language.

## Loading information from cookies

Now we have some information in a cookie waiting for our application to use it. The best moment to utilize such information is while creating a new session so the application could load preferences from the cookie and store them in the user session object. So let’s play with the method in WebApplication class which creates a new user session:

```java
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

So far so good, but the most important things happen in trySetLanguageFromCookie() method:

```java
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

In this method we first try to find **(1)** a cookie named LANGUAGE_COOKIE_NAME and if we find it, we modify **(2)** the session locale according to data loaded from the cookie. And another thing we could do is refresh **(4)** the just loaded cookie so if the user won’t click the flag for a long time (as he doesn’t have to because we automatically set the proper locale based on the cookie), he won’t lose his preference.

Now, everything should work fine. The user can choose the language when he enters our application for the first time and later we load this information from the cookie, set the proper locale and refresh the cookie so it won’t expire.
