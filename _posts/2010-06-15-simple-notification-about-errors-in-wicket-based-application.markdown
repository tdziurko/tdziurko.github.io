---
layout: post
title: Simple notification about errors in Wicket-based application
description: Sending an e-mail every time an unhandled exception appears in Apache Wicket application
date: 2010-06-15
author: tomek
image: '/images/pexels-wicket-errors.jpg'
tags: [Programming, Java]
tags_color: '#5fb05f'
redirect_from:
  - /2010/06/simple-notification-about-errors-in-wicket-based-application/
---

Today I am going to describe a simple and quick trick in Apache Wicket which allows our web application to send an email every time when something goes wrong.

Adding some method when exception appears in Wicket-based application is a very easy task. In the class extending WebApplication we should create our own implementation of method creating custom RequestCycle object where we override onRuntimeException method:

```java
@Override
public RequestCycle newRequestCycle(Request request, Response response) {
  return new WebRequestCycle(this, (WebRequest)request, (WebResponse)response) {

    @Override
    public Page onRuntimeException(Page page, RuntimeException exception) {
      //
      // do something with this exception
      //
      return super.onRuntimeException(page, exception);
    }
  };
}
```

This method is executed by the object responsible for request cycle processing named (what a surprise! 🙂 ) RequestCycleProcessor when any unhandled exception appears and allows us to return the Page class where we want to redirect the user after an error. But in this scenario we are going to use this method for a completely different purpose: sending an e-mail message with detailed information about the exception.

In this example to send emails I will use a Spring framework module which can be configured in the following way:

a) applicationContext.xml fragment:

```xml
<context:property-placeholder location="classpath*:application.properties" />

<bean id="mailSender">
  <property name="host" value="${smtp.host}" />
  <property name="port" value="${smtp.port}" />
  <property name="username" value="${smtp.user}" />
  <property name="password" value="${smtp.password}" />
  <property name="defaultEncoding" value="UTF-8" />
</bean>

<bean id="mailFactory">
  <constructor-arg value="${mail.fromName}"/>
  <constructor-arg value="${mail.fromAddress}"/>
  <constructor-arg value="${mail.defaultToAddress}"/>
</bean>
```

The second bean is a simple messages factory which builds emails with content. Its constructor needs some variables from the properties file.

b) WebApplication class fragment:

```java
public class ClientNotifierApplication extends WebApplication {

  private JavaMailSenderImpl mailSender;
  private MailMessageFactory mailMessageFactory;

  @Override
  protected void init() {
    super.init();

    mailSender = (JavaMailSenderImpl) getSpringContext().getBean("mailSender");
    mailMessageFactory = (MailMessageFactory) getSpringContext().getBean("mailFactory");
  }

  // ...

  @Override
  public RequestCycle newRequestCycle(Request request, Response response) {
    return new WebRequestCycle(this, (WebRequest) request, (WebResponse) response) {

      @Override
      public Page onRuntimeException(Page page, RuntimeException exception) {

        MimeMessage message = mailSender.createMimeMessage();
        mailMessageFactory.prepareWicketExceptionMessage(message, page, exception);
        mailSender.send(message);

        return super.onRuntimeException(page, exception);
      }
    };
  }
}
```

Because we passed the Page object to the method prepareWicketExceptionMessage, we have access to more detailed data about application state when the error appeared. We could read application name (useful when we use this mechanism in many projects), page parameters and through Session object also info about the logged user. It’s good to know to whom every error happened in case we would like to contact him and ask some questions about what he did, etc.

