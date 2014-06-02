---
author: Tomasz Dziurko
comments: true
date: 2010-06-28 18:31:59+00:00
layout: post
slug: solving-com-mysql-jdbc-exceptions-jdbc4-communicationsexception-in-spring-jdbc-based-application
title: Solving com.mysql.jdbc.exceptions. jdbc4.CommunicationsException in Spring
  JDBC based application
wordpress_id: 203
keywords: com.mysql.jdbc.exceptions. jdbc4.CommunicationsException, CommunicationsException , solution, Spring, JDBC]
categories:
- How-to
- Java
tags:
- CommunicationsException
- DBCP
- How-to
- Java
- JDBC
- Spring
---

Last week after releasing first version of Wicket, Spring and JDBC based  application into production I noticed strange behavior. Everyday first  attempt to enter the service caused unexpected exception. Quick glance  at Tomcat logs showed code presented below:

``` java
Caused by: com.mysql.jdbc.exceptions.jdbc4.CommunicationsException: The last packet successfully received from the server was156378 seconds ago.The last packet sent successfully to the server was 156378 seconds ago, which  is longer than the server configured value of 'wait_timeout'. You should consider either expiring and/or testing connection validity before use in your application, increasing the server configured values for client timeouts, or using the Connector/J connection property 'autoReconnect=true' to avoid this problem.
        at sun.reflect.NativeConstructorAccessorImpl.newInstance0(Native Method)
        at sun.reflect.NativeConstructorAccessorImpl.
newInstance(NativeConstructorAccessorImpl.java:39)
        at sun.reflect.DelegatingConstructorAccessorImpl.
newInstance(DelegatingConstructorAccessorImpl.java:27)
        at java.lang.reflect.Constructor.newInstance(Constructor.java:513)
        at com.mysql.jdbc.Util.handleNewInstance(Util.java:406)
        at com.mysql.jdbc.SQLError.createCommunicationsException(SQLError.java:1074)
        at com.mysql.jdbc.MysqlIO.send(MysqlIO.java:3246)
        at com.mysql.jdbc.MysqlIO.sendCommand(MysqlIO.java:1917)
        at com.mysql.jdbc.MysqlIO.sqlQueryDirect(MysqlIO.java:2060)
        at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2536)
        at com.mysql.jdbc.ConnectionImpl.execSQL(ConnectionImpl.java:2465)
        at com.mysql.jdbc.StatementImpl.executeQuery(StatementImpl.java:1383)
        at com.mysql.jdbc.ConnectionImpl.getTransactionIsolation(ConnectionImpl.java:3117)
        at org.apache.commons.dbcp.DelegatingConnection.
getTransactionIsolation(DelegatingConnection.java:313)
        at org.apache.commons.dbcp.PoolingDataSource
$PoolGuardConnectionWrapper.getTransactionIsolation(PoolingDataSource.java:239)
        at org.springframework.jdbc.datasource.DataSourceUtils.
prepareConnectionForTransaction(DataSourceUtils.java:173)
        at org.springframework.jdbc.datasource.DataSourceTransactionManager
.doBegin(DataSourceTransactionManager.java:210)
        ... 41 more
```

It looks that after few hours at night when no-one used the application,  MySQl server closed connection and first database query caused  exception. I double checked my connection url and settings in context.xml file to found nothing obviously wrong:<!-- more -->

``` xml
<Resource auth="Container" driverClassName="com.mysql.jdbc.Driver"
 factory="org.apache.commons.dbcp.BasicDataSourceFactory"
   logAbandoned="true" maxActive="32" maxIdle="1" name="myDB"
   password="secret" removeAbandoned="true" removeAbandonedTimeout="300"
   type="javax.sql.DataSource"
   url="jdbc:mysql://mywebapp:3306/myDB?autoReconnect=true&characterEncoding=utf-8&useUnicode=true" username="user"
 />
```

I had used the same settings in pure JDBC projects few times before and  never encoutered similar exception so now I wanted to ask uncle Google  for some answers :) At the beginning I dove into the MySQL manual to  check why autoReconnect isn't  enough in my application but found nothing relevant to my problem. After  some thinking I got the idea that maybe some settings in Commons-DBCP  are missing so I read [dbcp configuration](http://commons.apache.org/dbcp/configuration.html) to find that connection could be validated before real usage by specifing two parameters: testOnBorrow="true" validationQuery="select 1"  . As written in docs:


<blockquote>testOnBorrow - The indication of whether objects will be validated  before being borrowed from the pool. If the object fails to validate, it  will be dropped from the pool, and we will attempt to borrow another.
NOTE - for a true value to have any effect, the validationQuery parameter must be set to a non-null string.
validationQuery  - The SQL query that will be used to validate connections from this  pool before returning them to the caller. If specified, this query MUST  be an SQL SELECT statement that returns at least one row.</blockquote>


After adding these two parameters to context.xml my problems dissapeared.
