---
author: Tomasz Dziurko
comments: true
date: 2009-05-10 12:15:02+00:00
layout: post
slug: java-persistence-api-and-simple-audit
title: Java Persistence API and simple audit
wordpress_id: 140
categories:
- How-to
- Java
tags:
- audit
- How-to
- Java
- JPA
- PrePersist
- PreUpdate
---

Recently for a first time  have to create in the application functionality to track changes in the database made by users. There are some existing solutions and one of I found using Google is very interesting article at JavaLobby ["Using a Hibernate Interceptor to set audit trail properties"](http://java.dzone.com/articles/using-a-hibernate-interceptor-). But there is a much simpler solution which I present here.

If we are using Java Persistence API we can pick very useful annotations: @PrePersist and @PreUpdate. Suppose that in every class we want to gather data about who and when created database record and who and when edited it for a last time. Because such functionality is needed in every entity class we can move logic to common abstract class which will be extended by all concrete entity classes:<!-- more -->

[java]
<pre>import java.util.Date;
import javax.persistence.*;

@MappedSuperclass
public abstract class BaseEntity {

 private String createdBy;
 private Date createdOn;
 private String modifiedBy;
 private Date modifiedOn;

 @Column(name="created_by")
 public String getCreatedBy() {
  return createdBy;
 }

 @Column(name="created_on")
 @Temporal(TemporalType.TIMESTAMP)
 public Date getCreatedOn() {
  return createdOn;
 }

 @Column(name="modified_by")
 public String getModifiedBy() {
  return modifiedBy;
 }

 @Column(name="modified_on")
 @Temporal(TemporalType.TIMESTAMP)
 public Date getModifiedOn() {
  return modifiedOn;
 }

 // getters and setters omitted

 @PrePersist
 public void prePersist() {
  User user = ApplicationUtils.getLoggedUser();
                setCreatedBy(user.getName());
  setCreatedOn(new Date());
 }

 @PreUpdate
 public void preUpdate() {
  User user = ApplicationUtils.getLoggedUser();
  setModifiedBy(user.getName());
  setModifiedOn(new Date());
 }
}
</pre>
[/java]

Methods marked by annotations are called just before saving changes in database so we can add our information about user and date of operation.

All annotations with similar effect existing in JPA are: PostLoad, PostPersist, PostRemove, PostUpdate, PrePersist, PreRemove, PreUpdate. Thanks to them we could be informed about all read/write/delete operations in our application. This can be very useful when in example we want to add such data to log files.
