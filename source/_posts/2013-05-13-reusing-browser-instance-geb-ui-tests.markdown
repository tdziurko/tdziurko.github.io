---
author: Tomasz Dziurko
comments: true
date: 2013-05-13 10:46:19+00:00
layout: post
slug: reusing-browser-instance-geb-ui-tests
title: Reusing browser instance in Geb UI tests
wordpress_id: 1833
categories:
- How-to
tags:
- geb
- groovy
- selenium
- UI tests
---

<!-- more -->

#### UPDATE


_When I originally wrote this post, I was Geb newbie. Despite the fact that approach presented below is valid and works, there is a much simpler solution to this problem incorporated by Geb itself and I strongly suggest using it instead of mine, home-crafted hack. All credits go to [Artur Gajowy](http://stackoverflow.com/users/52142/artur-gajowy) for showing me better approach.

I have added short paragraph about this simpler solution at the end of this article, probably you should go [directly there](<a href="/2013/05/reusing-browser-instance-geb-ui-tests/#better-solution) :).
_




 
[Geb](http://gebish.org) is next-generation UI testing library that frees developer from dealing with Selenium API which is in many cases not so friendly. The most important feature of Geb is its support, or I would even say, strong encouragement to use Page Objects in your UI tests. Page Objects are also present in Selenium, but they are not as intuitive and east to use as in Geb.
But enough about Geb itself. Today I will share with you simple trick how to share one browser instance when running many Geb UI tests. This will significantly reduce time needed to execute UI tests and also reduce memory usage on test machines.


### The Problem


Assume we have many test classes (spefications) similar to this one below.

``` groovy
class ExampleUITest extends GebSpec {

    def "Should test feature 1"() {
        given:
        to LoginPage

        when:
        loginWith("login", "password")

        then:
        at HomePage
        assert loggedUserLabel == "loggedUser"
    }

    def "Should test feature 2"() {
        given:
        to LoginPage

        when:
        loginWith("login", "password")

        then:
        at HomePage
        assert loggedUserLabel == "loggedUser"
    }
   
    // and so on...
```

What we have to do is prepare browser instance (in our case Firefox one) to click through our tests. We could to it in setup() method:

``` groovy
class ExampleUITest extends GebSpec {

    def setup() {
        driver = new FirefoxDriver()
    }

    def "Should test feature 1"() {
        // ...
    }  

    def cleanup() {
        driver.quit()
    }
```

This approach works but it has one, big drawback. We all know that UI tests tend to last quite long and creating new browser instance for each, yes, each test method will only make this situation worse as it also takes some time to start browser on your testing server.

**Second approach**

We also could reuse same browser in each specification (each Geb class) by using setupSpec() and cleanupSpec() methods instead. This will reduce number of new instances created but still we will have many of them, especially in large codebase with many test classes.



### Solution


My solution to this issue takes from both approaches presented above. But instead of creating browser for each specification, I am reusing one created in the first executed class.

To make it work we need to introduce common abstract class for all our UI tests:
``` groovy
class IntelliGebSpec extends GebSpec {

    static def cachedDriver // static variable will store our single driver instance

    def setupSpec() {
        if ( cachedDriver == null) { // in first Specification this will be true
            cachedDriver = new FirefoxDriver()  // so let's create driver and put it in cachedDriver variable
        }
    }

    def setup() {
        driver = cachedDriver   // each test should use our cached browser instance
    }

    // as we are reusing browser, we should logout from our application after
    // each test to have clean browser state on each test run
    def cleanup() {  
        if ($(id: "logout").isDisplayed()) {
            $(id: "logout").click()
            waitFor { $(id: "inputName").isDisplayed() }
        }
        driver.manage().deleteAllCookies()
    }
}    
```  



### Summary


In this post I have shown my home made solution to reusing browser in multiple tests. I strongly suggest all of you to check and try Geb, as after over a year of using Selenium I am really impressed by this library. Easy to use with really cool Page Objects support.



### Update: the best solution


As explained in reference documentation [here](http://www.gebish.org/manual/current/configuration.html#driver_caching), Geb provides mechanism to configure and reuse single instance of browser in all our UI tests. Only thing we have to do is create GebConfig script in default package that will contain driver instantiation script, e.g.:
``` groovy
// complete GebConfig.groovy file:
driver =
    {
        newDriver = new FirefoxDriver()
        newDriver.manage().window().setSize(new Dimension(1024, 768))
        return newDriver
    }
```

And that's it, no hacking, no ugly static field.
