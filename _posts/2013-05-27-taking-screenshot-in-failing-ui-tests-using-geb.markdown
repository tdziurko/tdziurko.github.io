---
layout: post
title: Taking screenshot in failing UI tests using Geb
description: Configuring Geb and Spock to automatically take a browser screenshot whenever a UI test fails
date: 2013-05-27
author: tomek
image: '/images/geb-screenshot-cover.jpg'
tags: [Programming, Java]
tags_color: '#5fb05f'
---

There is nothing worse than being reported that a UI test failed but without clear evidence of what exactly went wrong. And what could be better evidence than a screenshot just after the failure?

This is a second post about [**Geb**](http://www.gebish.org/) in a short time. This time we will learn how to configure Geb to take a screenshot every time our UI test fails. An image of the page at the moment when the test failed is often very helpful in debugging a mysteriously failing UI test on a CI server and can save a lot of time when trying to solve such problems.

### The Problem

We need a simple way to configure Geb UI tests to take a screenshot on each test fail. This should be “add and forget” config that does not need any changes when we add more tests to our application.

### Solution

We have to find some type of listener that waits for different test results and fires the proper method when a test fails. In [TestNG](http://testng.org/) we have a [TestListenerAdapter](http://testng.org/javadocs/org/testng/TestListenerAdapter.html) so probably Geb or Spock should have something similar.

After some digging I was able to locate _IRunListener_ with the method I need:

```groovy
public interface IRunListener {

    void beforeSpec(org.spockframework.runtime.model.SpecInfo specInfo);

    void beforeFeature(org.spockframework.runtime.model.FeatureInfo featureInfo);

    void beforeIteration(org.spockframework.runtime.model.IterationInfo iterationInfo);

    void afterIteration(org.spockframework.runtime.model.IterationInfo  iterationInfo);

    void afterFeature(org.spockframework.runtime.model.FeatureInfo featureInfo);

    void afterSpec(org.spockframework.runtime.model.SpecInfo specInfo);

    void error(org.spockframework.runtime.model.ErrorInfo errorInfo); // < -- this is it

    void specSkipped(org.spockframework.runtime.model.SpecInfo specInfo);

    void featureSkipped(org.spockframework.runtime.model.FeatureInfo featureInfo);
}
```

**Simple version of test failure listener**
Luckily this interface comes with an abstract class named _AbstractRunListener_ that allows us to implement only methods we want instead of fulfilling the whole contract specified by IRunListener.

So our first step will be to print something to the console every time our test fails:

```groovy
class TakeScreenshotOnTestFailureListener extends AbstractRunListener {

   def void error(ErrorInfo error) {
       System.out.println("Error in method " + error.method.name);
   }
}
```

**Adding custom listener to all tests with Spock extension**
But apart from implementing a listener, we have to add this listener to our tests. Currently in Spock this is possible only with extensions. Again, I had to find the proper extension to use and ended up with the interface IGlobalExtension which has only one method visiting each spec (test method) in our codebase:

```groovy
public interface IGlobalExtension {
  void visitSpec(SpecInfo spec);
}
```

So extension implementation looks as follows, to each visited spec we add our TakeScreenshotOnTestFailureListener listener:

```groovy
class GlobalSpecExtension implements IGlobalExtension {

    @Override
    void visitSpec(SpecInfo specInfo) {
        specInfo.addListener(new TakeScreenshotOnTestFailureListener())
    }
}
```

**Registering Spock global extension**

Each global extension has to be registered to Spock. This can be achieved by placing a text file with easy to remember name in META-INF/services directory. The file must be named _org.spockframework.runtime.extension.IGlobalExtension_ and should contain the full name of our implementation of IGlobalExtension interface:

```text
com.smscpl.mc5.uitests.geb_extensions.GlobalSpecExtension
```

**Taking a screenshot on each UI test fail**
Now on each test fail we will see that our text message is printed to the console. This proves that our solution is going in the right direction and the last thing still not implemented is taking a screenshot.

Taking a screenshot using Selenium driver is quite easy, it is one method call, but before that we have to acquire driver instance in our listener class. It is impossible to access the driver directly from the listener so we need to store it in a globally accessible object. The simplest solution is to save a reference to the driver when we create it in our GebConfig file. So let’s create a singleton that will hold our browser:

```groovy
@Singleton
class BrowserInstance {
    def FirefoxDriver browser;
}
```

and initialise it in GebConfig.groovy script located in the root package of our project:

```groovy
driver =
    {
        newDriver = new FirefoxDriver()
        BrowserInstance.instance.browser = newDriver
        return newDriver
    }
```

Now we can access the driver from our listener object, so the final taking screenshot logic is pretty straightforward:

```groovy
class TakeScreenshotOnTestFailureListener extends AbstractRunListener {

   def void error(ErrorInfo error) {
       File screenFile = BrowserInstance.instance.browser.getScreenshotAs(OutputType.FILE); // take a screenshot

       String fileName = createFileName(error) // generate file name based on test class and method names
       File destinationFile = new File(System.getProperty("java.io.tmpdir") + System.getProperty("file.separator") + fileName)
       Files.copy(screenFile, destinationFile)

       System.out.println("##teamcity[publishArtifacts '" + destinationFile.getAbsolutePath() + "']"); // publish as artifact in TeamCity
   }

    def private String createFileName(ErrorInfo error) {
        String specName = error.method.parent.name
        String testName = error.method.name
        String milliseconds = new Date().getTime()
        String fileName = specName + ": '" + testName + "' " + milliseconds + ".png"
        fileName
    }
}
```

And that’s all. Each failed test will now appear with corresponding screenshot from the browser.
