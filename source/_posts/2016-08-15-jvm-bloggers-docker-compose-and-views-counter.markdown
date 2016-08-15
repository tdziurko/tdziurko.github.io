---
layout: post
title: What's new in JVM Bloggers - docker compose and views counter 
date: 2016-08-15 10::00:00 +0200
comments: true
categories: 
- jvm-bloggers
tags:
- docker
- akka
- jvm-bloggers
keywords: 
description: New things in JVM Bloggers in version 0.10.0 - docker compose, views counter  
---

Version 0.10.0 is live with Docker Compose and views counter

<!-- more -->

As some of you may be aware, for the last year I am working on a pet project called [JVM Bloggers](https://github.com/tdziurko/jvm-bloggers/) with a goal 
to popularize blogging and knowledge sharing in the Polish Java developers community. We send a weekly summary with new blog posts and new videos published in the last 7 days. 


During last few weeks two major changes were added to the project: 

* deployment is done using Docker Compose

* All clicks are going through the application so we can gather some statistics   

 
# Docker Compose
 
Previously application was deployed using two separate Dockers, that was the easier approach to get things going 
but as we are moving to microservices architecture I knew that Docker Compose will be a good choice allowing us 
to easily add more and more elements to our system.

First, [docker-compose.yml](https://github.com/tdziurko/jvm-bloggers/blob/b1e243ffd2bdb98d1075c0ded51612a18bd7a088/docker-compose.yml) needs to be created with two services: jvm_bloggers_db and jvm-bloggers-core:

```
version: '2'
services:
  jvm_bloggers_db:
    image: sameersbn/postgresql:9.4-22
    environment:
      DB_USER: $JVM_BLOGGERS_DB_USER
      DB_PASS: $JVM_BLOGGERS_DB_PASSWORD
      DB_NAME: $JVM_BLOGGERS_DB_NAME
    ports:
      - "$JVM_BLOGGERS_DB_PUBLISHED_PORT:5432"
    volumes:
      - "$JVM_BLOGGERS_DB_PATH:/var/lib/postgresql"
  jvm-bloggers-core:
    image: tdziurko/jvm-bloggers:$JVM_BLOGGERS_CORE_IMAGE_VERSION
    environment:
      spring.profiles.active: $JVM_BLOGGERS_CORE_SPRING_PROFILES
      jasypt.encryptor.password: $JVM_BLOGGERS_CORE_ENCRYPTOR_PASSWORD
    ports:
      - "$JVM_BLOGGERS_CORE_PORT:8080"
    volumes:
      - /etc/localtime:/etc/localtime:ro
      - /usr/share/zoneinfo/Europe/Warsaw:/etc/timezone:ro
      - /tmp
    links:
      - jvm_bloggers_db
    depends_on:
      - jvm_bloggers_db
```

In _jvm_bloggers_db_ we have some parameters that allows to customize PostgreSQL image, besides most obvious ones like database name, user and password there are also:

* _$JVM_BLOGGERS_DB_PUBLISHED_PORT_ - port on which our database will be available from external (like developer's machine)
 
* _$JVM_BLOGGERS_DB_PATH_ - path to a directory where database files are located on our server

_jvm-bloggers-core_ is more complicated:
   
* environment variables specifies Spring profile and master password used to encrypt properties like API keys, password to admin panel, etc.
   
* _$JVM_BLOGGERS_CORE_PORT_ - port on which application is available, on production it is obviously 80, but locally it can be different   

* link _jvm_bloggers_db_ allows us to use this as a url to the database in _spring.datasource.url_ with value _jdbc:postgresql://jvm_bloggers_db:5432/jvm_bloggers_prod_ without worrying where exactly database container is running in our internal network.
   

Next to docker-compose file we need a [bash script](https://github.com/tdziurko/jvm-bloggers/blob/b1e243ffd2bdb98d1075c0ded51612a18bd7a088/jvm-bloggers.sh) to start/stop our system:
  
``` bash
#!/bin/sh

setupProperties() {
  # Database settings
  export JVM_BLOGGERS_DB_USER=jvm_bloggers
  export JVM_BLOGGERS_DB_PASSWORD=jvm_bloggers
  export JVM_BLOGGERS_DB_NAME=jvm_bloggers
  export JVM_BLOGGERS_DB_PATH="~/postgresql-data/"
  export JVM_BLOGGERS_DB_PUBLISHED_PORT=5432

  # Core Application settings:
  export JVM_BLOGGERS_CORE_IMAGE_VERSION=0.9.0-20160724-165001-19f0d70
  export JVM_BLOGGERS_CORE_SPRING_PROFILES=dev
  export JVM_BLOGGERS_CORE_ENCRYPTOR_PASSWORD=secret
  export JVM_BLOGGERS_CORE_PORT=9000
}

start() {
    docker-compose up -d
    echo "Started JVM Bloggers"
}

stop() {
    docker-compose down
}

case "$1" in
  start)
    start
    ;;
  stop)
    stop
    ;;
  status)
    status
    ;;
  restart)
    stop
    start
    ;;
  *)
    echo "Usage: $0 {start|stop|restart|status}"
esac  

```
  

Main part of this file consists of multiple exports that enable customization of docker-compose so it can be 
used without changes locally and on production. Then we have simple commands to launch, shutdown and 
check status of running services. And that's all, now we can use following command to launch all services:
 
``` bash
./jvm-bloggers.sh start 
```

    
# Views counter
  
Next feature released with 0.10.0 version is views counter. I wanted to have some insight into 
how many people use JVM Bloggers and, most importantly, what are the most popular entries. In next 
weeks we plan to add section with most popular posts from last month so authors of most valuable 
content could get extra recognition.
   
To add this feature following things were required:
   
1. Redirect controller

2. Migration from UUID as a unique blog post identifier to something shorter

3. Logic to store clicks in the database


Job of [Redirect Controller](https://github.com/tdziurko/jvm-bloggers/blob/24d00aad8f90c5e13087ac91e5db0c9f7fd3fe2a/src/main/java/pl/tomaszdziurko/jvm_bloggers/rest/blogpost_redirect/RedirectController.java) is very simple - get UID from the url parameter (e.g. http://jvm-bloggers.com/r/wx1LivZ) 
and redirect user to proper blog post. But before 0.10.0 UID was a full 

``` java
_UUID.randomUUID().toString()_ 
``` 

so instead of a nice and short link we had something like http://jvm-bloggers.com/r/b877a837-4f5b-4a0b-90b2-981c83c1366e, 
not very friendly and concise. Simple data migration was necessary and in a new approach UID is compact 7-chars length String:
   
``` java
RandomStringUtils.randomAlphanumeric(7);
```   

The last thing we needed was a logic to save clicks. I didn't want to make a redirection slower only because 
we are storing something in the database so I decided to use Akka to handle this work in a separate thread.

In RedirectController we publish SingleClick event:

``` java
actorRef.tell(new SingleClick(blogPost.get()), noSender());
```

and actors simply handles this event by persisting Click entity:

``` java

@Slf4j
public class ClicksStoringActor extends AbstractActor {

    public ClicksStoringActor(ClickRepository clickRepository, NowProvider nowProvider) {
        log.debug("Creating " + ClicksStoringActor.class.getSimpleName());
        receive(ReceiveBuilder.match(SingleClick.class,
            clickEvent -> {
                log.debug("Storing click for " + clickEvent.getBlogPost().getUrl());
                clickRepository.save(new Click(clickEvent.getBlogPost(), nowProvider.now()));
            }).build()
        );
    }

    public static Props props(ClickRepository clickRepository, NowProvider nowProvider) {
        return Props.create(ClicksStoringActor.class, () -> {
                return new ClicksStoringActor(clickRepository, nowProvider);
            }
        );
    }
}


```


### Credits

These new features wouldn't be possible without contributors, most notably [Mateusz Urbański](https://github.com/matek2305) and [Marcin Kłopotek](https://github.com/goostleek).
 
# Summary

JVM Bloggers is still evolving with both small changes and major new features, so if you want to help 
please contact me or join our [gitter](https://gitter.im/tdziurko/jvm-bloggers) channel. We have Java8, Spring Boot, Docker, 
Akka on board and plenty of ideas to implement (even using different languages). 

For more details you should visit our [GitHub repository](https://github.com/tdziurko/jvm-bloggers).  