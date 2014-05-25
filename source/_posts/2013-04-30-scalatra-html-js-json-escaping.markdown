---
author: Tomasz Dziurko
comments: true
date: 2013-04-30 21:59:59+00:00
layout: post
slug: scalatra-html-js-json-escaping
title: 'Easy Html/Javascript JSON escaping in Scalatra '
wordpress_id: 1802
categories:
- How-to
- Scala
tags:
- Html escaping
- scala
- Scalatra
---
<!-- more -->
[![logo-scalatra](/images/blog/2013/04/logo-scalatra.png)](/images/blog/2013/04/logo-scalatra.png)
[Scalatra](http://www.scalatra.org/) is a small web-framework that allows to easily build Rest API for web applications. We are using it in two projects to serve JSON data consumed by AngularJS based front-ends. And in one of this project we had a problem with text messages that could contain Html/Javascript elements. User could save message and then this message could be used on two pages. On first we need to display escaped data (read-only view) and on second (edit page) we wanted to display message in the exactly same way as user typed it.
So basically we needed easy and declarative way to define which Scalatra Rest end-points should return escaped and not-escaped JSON data.



### Basic Scalatra project


To show how it could be accomplished, we need a simple sample project using Scalatra. I didn't want to create it from scratch by myself so I removed some Swagger related stuff from project by my colleague Krzysztof Ciesielski ([his post about Scalatra and Swagger](http://abstractionextraction.wordpress.com/2013/03/31/using-swagger-with-scalatra/)) and I was ready to go :) Full commit with this basic project is available [here](https://github.com/tdziurko/scalatra-json-escaping/commit/099c7ac30fe0f63319a890e848a94390c035e801), but the most interesting class is shown below:
<!-- more -->
[scala]
class ExampleServlet() extends ScalatraServlet with JacksonJsonSupport with JValueResult {

  protected implicit val jsonFormats: Formats = DefaultFormats

  val messages = List(
    Message(1, "<script><alert>Script hacker</alert></script>", "Johny Hacker", new Date()),
    Message(2, "Decent message", "John Smith", new Date()),
    Message(3, "Message with <b>bolded fragment</b>", "Kate Norton", new Date())
  )

  before() {
    contentType = formats("json")
  }

  get("/") {
    messages
  }

  get("/:id") {
    val id = params("id").toInt;
    val messageOptional = messages.find((m: Message) => m.id == id)

    log("optional =" + messageOptional)
    messageOptional match {
      case Some(e) => e
      case _ =>
    }
  }
}

case class Message(id: Int, text: String, author: String, created: Date)
[/scala]
Ok, so what we have here? Simple Rest service that returns JSON. Get method without any params returns list of messages and get with passed id returns single message with given id. For the sake of simplicity, I have hardcoded list of three messages with two of them containing some Html/JS elements.



### Definition of Done


It is always good to have clear Definition of Done so in our case there are two things we want our methods to do:



  
  * Get /rest/example without parameters should return list of messages with escaped content

  
  * Get /rest/example/1 (with parameter) should return single messages without escaping





### Implementation details


To add escaping we need to find place in Scalatra where objects returned from method are converted to Json. After some digging I found _JacksonJsonOutput_ trait with method _writeJon_:
[scala]
trait JacksonJsonOutput extends JsonOutput[JValue] with jackson.JsonMethods {
  protected def writeJson(json: JValue, writer: Writer) {
    mapper.writeValue(writer, json)
  }
}
[/scala]

So what we have to do is override method and somehow define logic to perform actual escaping for us. Luckily JValue has function _map_ that allows to apply passed function to every value in JSON. So we could fire escapeHtml4 method from Apache Commons Lang on each String value in JSON:
[scala]
  override def writeJson(json: JValue, writer: Writer) {
    val escapedJson = json.map((x: JValue) =>
      x match {     // let's check what type is this element
        case JString(y) => JString(escapeHtml4(y))    // if it is a String, escape it
        case _ => x        // keep value of other type as is
      }
    )
    mapper.writeValue(writer, escapedJson)
  }
[/scala]

As you can see all logic is done by _map_ function, we only need to add a few lines of code that will filter and escape String values in JSON. So far so good, but now we are escaping every methods from our Rest API. This is not exactly what we wanted to achieve. To get it done right, we need to define a mechanism to disable escaping for a specific method.



#### Wrapping marker object


So we need a marker "something" (class, interface or trait) that will tell _writeJson_ method which data shouldn't be escaped.
And after trying different approaches it turned out that only thing we need is to wrap value returned from method in a marker object and then in _writeJson_ check if its is wrapped or not and escape only not wrapped data. So workflow looks as follows:




  1. If we dont' want escaping, wrap returned object in a _NotEscapedJsonWrapper_


  2. In writeJson check if object to write is a wrapper. If so, only unwrap it and write. It it is not a wrapper, escape all Strings inside object and wride data after that



Looks easy. We need simple wrapper class:
[scala]
  case class NotEscapedJsonWrapper[T](notEscapedData: T)
[/scala]

and we need to wrap response with this object:
[scala]
  get("/:id") {
    val id = params("id").toInt;
    val messageOptional = messages.find((m: Message) => m.id == id)

    log("optional =" + messageOptional)

    messageOptional match {
      case Some(e) => NotEscapedJsonWrapper(e) // wrap data that we don't want to be escaped
      case _ =>
    }
  }
[/scala]

And after that we have to modify _writeJson_ to check if object to be written is wrapped with "not-escape-me" marker object or not.
[scala]
  override def writeJson(json: JValue, writer: Writer) {
    (json \ "notEscapedData") match {     // check if json contains field 'notEscapedData' meaning that it is wrapping object
      case JNothing => {                  // no wrapper, so we perform escaping
        val escapedJson = json.map((x: JValue) =>
          x match {
            case JString(y) => JString(escapeHtml4(y))
            case _ => x
          }
        )
        mapper.writeValue(writer, escapedJson)
      }
      case _ => {    // field 'notEscapedData' detected, unwrap and return clean object
        mapper.writeValue(writer, json \ "notEscapedData")
      }
    }
  }
[/scala]

And after that we could compile and start container with our small application. When we enter /rest/example we will get list of all messages with every String properly escaped. And if we request /rest/example/1 we will get single message as not-escaped data. That's all, we have our escaping working.



### Summary


In this short post I've described how to implement Html/Js escaping in Scalatra based application in a pretty straightforward way that can be easily defined in one, common place without need to add manual escaping in every method that needs that. Thanks to this, our application is much safer and only explicitly declared methods can return potentially unsafe data.
Source code of this small example app is available [on GitHub](https://github.com/tdziurko/scalatra-json-escaping).
 
