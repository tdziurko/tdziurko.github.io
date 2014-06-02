---
author: Tomasz Dziurko
comments: true
date: 2013-02-28 22:26:25+00:00
layout: post
slug: twitter-bootstrap-navbar-angularjs-component
title: Twitter Bootstrap Navbar as AngularJS component
wordpress_id: 1726
keywords: twitter, bootstrap, navbar, angularJS
categories:
- How-to
- JavaScript
tags:
- angularjs
- Bootstrap
---

You all know [Twitter Bootstrap](http://twitter.github.com/bootstrap/), don't you? It's the awesome library to make your web application looks pretty good without spending many hours on CSS. We are using Bootstrap based design in our current project that also uses AngularJS and as we are planning to have many new pages, we decided that's the earlier we introduce some components to our UI the better. And, what shouldn't be a surprise, navigation bar natually became the first candidate.


### Overview


So in this post I am going to show how externalize Twitter Bootstrap Navbar into a separate, flexible AngularJS component. All source code is available in [my GitHub repository](https://github.com/tdziurko/twitter-bootstrap-navbar-as-angularjs-directive) , so everything should be easy to follow and repeat.


### Basic setup


To start from the basics, I've prepared three pages that use the same navigation bar. These pages will soon become our super simple web application using AngularJS:


->![navbar_angularjs1](/images/blog/2013/02/navbar_angularjs1.png)<-


Next step will be adding AngularJS and converting three pages into one small application. This is done in this [commit](https://github.com/tdziurko/twitter-bootstrap-navbar-as-angularjs-directive/commit/9d135332ef5e934920a65975f41fb06f6cc852a8). And after that we are ready to start working on our Navbar component.<!-- more -->


### Problem


On each page we have very similar html fragment rendering navbar element.

``` html
    <div class="navbar navbar-inverse">
        <div class="navbar-inner">
            <div class="container">
                <a class="brand">Example App</a>

                <div class="nav-collapse">
                    <ul class="nav">
                        <li class="active"><a href="home.html">Home</a></li>
                        <li><a href="second.html">Second</a></li>
                        <li><a href="third.html">Third</a></li>
                    </ul>
                </div>
            </div>
        </div>
    </div>
```

The only difference is in where class _active_ which highlights link is located. And every time we see such repetition, our DRY detector should start ringing and don't stop until we do something about it.


### Simple directive


For a start, we will move this code fragment into a AngularJS directive. At first, it will be only a "dumb" piece of code that just renders the same html in different places. All html from the listing above is now located in the components/bootstrapNavbar.html file.

Next thing we need is a directive declaration in JS file:
``` js
angular.module("navbarapp", ["controllers"])
  .directive("bootstrapNavbar", function() { // (1)
  return {
    restrict: "E",         // (2)
    replace: true,         // (3)
    transclude: true,      // (4)
    templateUrl: "bootstrapNavbar.html"    // (5)
  }});
;
```

What happens here:



   
  1. we declare directive, please keep in mind that name of directive you use in the code will be boostrap-navbar, camelCase is transformed into - character.

   
  2. restrict: E - means that directive is used as an element, there are other options: C-class, A-attribute, more on [docs](http://docs.angularjs.org/guide/directive)

   
  3. replace: true - means that markup from template will completely replace directive element in the page markup

   
  4. transclude: true - what it means... you better tell me, I am not able to describe it more vaguely that it is done in the [docs](http://docs.angularjs.org/guide/directive) ;)

   
  5. templateUrl - file where we store our template content



After that we should replace navbar html markup on every page with something like:

``` html
<bootstrap-navbar></bootstrap-navbar>
```

Now it's high time to test it. Unfortunately there is one problem. Security policy does not allow browser to execute XHR(Ajax) calls from html file stored on our disk. So we have to setup lightweight httpServer to serve our files to browser. You can do it using Jetty or anything similar. I tried one from Python and it worked without issues. So basically navigate to directory where all files are stored, execute:

``` python
python -m SimpleHTTPServer 9000
```

and then open your browser using address [http://localhost:9000/home.html](http://localhost:9000/home.html). You should see the same page as before, but now it is using our directive.



### Flexibility++


So we have our first directive, yay! But its IQ is not that high. If you navigate to Second and Third Page, you will notice that it always renders link to Home Page as active, which is not exactly true for all pages. And in this paragraph we will try to make it smarter.

At first, let's add attribute _current-tab_ to our directive usages:
``` html
  <bootstrap-navbar current-tab="Second"></bootstrap-navbar>
```

Passed value must be the same as link name in the navbar, so in our example app it should be either: Home, Second or Third.

Next step is to prepare our template file for easy modification. Because we will use jQuery selectors to find link to activate, we will remove active class and also add an id attribute to _ul_ element to make it easier to locate with selector:

``` html
<div class="navbar navbar-inverse">
    <div class="navbar-inner">
        <div class="container">
            <a class="brand">Example App</a>

            <div class="nav-collapse">
                <ul class="nav" id="navigation-tabs">     <!-- id added here -->
                    <li><a href="../home.html">Home</a></li>   <!-- removed class='active' -->
                    <li><a href="../second.html">Second</a></li>
                    <li><a href="../third.html">Third</a></li>
                </ul>
            </div>
        </div>
    </div>
</div>
```

**Core logic**
Now the last thing we have to do is some jQuery selector magic with little addition of AngularJS magic :)

Angular directives have function _compile_ which allows to alter directive template markup, so we will use it here. What this function receives is value of parameters defined on the component usage in our pages, so basically we have an access to all attributes that are declared where our navbar component is used, specifically our _current-tab_ attribute (again transformed to _currentTab_ inside JS file).
So when we know which link should be highlighted, jQuery selectors come to do the actual job:

``` js
angular.module("navbarapp", ["controllers"])
  .directive("bootstrapNavbar", function() {
  return {
    restrict: "E",
    replace: true,
    transclude: true,
    templateUrl: "components/bootstrapNavbar.html",
    compile: function(element, attrs) {  // (1)
      var li, liElements, links, index, length;

      liElements = $(element).find("#navigation-tabs li");   // (2)
      for (index = 0, length = liElements.length; index < length; index++) {
        li = liElements[index];
        links = $(li).find("a");  // (3)
        if (links[0].textContent === attrs.currentTab) $(li).addClass("active"); // (4)
      }
    }
  }});
;
```

So what we do here:



  
  1. define compile function that has passed two things, element on which it is called (template markup in our case) and attrs, an object with all attributes declared where our directive was used.

  
  2. select all list items from #navigation-tabs node

  
  3. find all links there, it will be only one

  
  4. if text content of link is equal to currentTab attribute we know that this is the link to highlight, so we simply add proper CSS class there



It's the simplest working solution. Of course we could add some error checking, but I wanted to show easy to follow approach to introducing flexible components to AngularJS applications.



### Summary


In this post I tried to show how with a few simple steps we could apply DRY rule to our html markup in AngularJS application. And as your application grow, you will probably find more and more places where html is duplicated making it a good candidate to convert it into component directive.
