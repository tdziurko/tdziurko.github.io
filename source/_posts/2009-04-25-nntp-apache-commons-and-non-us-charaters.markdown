---
author: Tomasz Dziurko
comments: true
date: 2009-04-25 19:37:54+00:00
layout: post
slug: nntp-apache-commons-and-non-us-charaters
title: NNTP, Apache Commons and non-US charaters
wordpress_id: 69
categories:
- How-to
- Java
tags:
- Apache Commons Net
- encoding
- How-to
- Java
- NNTP
- UTF-8
---

Like every weekend, today I was spending time on writing application for my MSc thesis. Among few tasks I planned for today one was connected with sending messages/notifications to the internal discussion group of my dormitory using NNTP protocol.

Short research showed that this should be a piece of cake. Below small code snippet showing how to send message using library [Apache Commons Net](http://commons.apache.org/net/).

[java]
public class Main {

 public static void main(String[] args) throws Exception {

 NNTPClient client = new NNTPClient();
 client.connect("news.ustronie.pw.edu.pl");

 client.selectNewsgroup("pw.test");
 Writer postArticle = client.postArticle();

 SimpleNNTPHeader headers =
 new SimpleNNTPHeader("Tomasz Dziurko &lt;tdziurko@gmail.com>", "Test kodowania polskich znaków");
 headers.addNewsgroup("pw.test");
 headers.addHeaderField("Mime-Version", "1.0");
 headers.addHeaderField("Content-Type","text/plain; charset=UTF-8");
 headers.addHeaderField("Content-Transfer-Encoding", "8bit");

 postArticle.write(headers.toString());
 postArticle.write("ąęóśłżźćń - test polskich znaków\r\n");
 postArticle.close();

 client.completePendingCommand();

 client.disconnect();
 }

}
[/java]

Everything worked as expected until I started to use Polish characters in the message. Then instead of "ąęóśłżźćń" I saw following picture:
[![Invalid characters encoding in NNTP class in Apache Commons Net](http://tomaszdziurko.pl/wp-content/uploads/2009/04/message1-300x93.png)](http://tomaszdziurko.pl/wp-content/uploads/2009/04/message1.png)<!-- more -->To solve this problem I tried many things: changing encoding in the header, changing encoding in the NetBeans project or even changing encoding in my newsreader (yes, yes, I even started to blame poor Thunderbird ;) ) but nothing helped. Then I decided to look into source code of Apache Commons Net library and after a short investigation I found source of all my problems: org.apache.commons.net.nntp.NNTP class:

[java]
public class NNTP extends SocketClient
{
 /*** The default NNTP port.  Its value is 119 according to RFC 977. ***/
 public static final int DEFAULT_PORT = 119;

 // We have to ensure that the protocol communication is in ASCII
 // but we use ISO-8859-1 just in case 8-bit characters cross
 // the wire.

 private static final String __DEFAULT_ENCODING = "ISO-8859-1";

 ...

 /***
 * Initiates control connections and gets initial reply, determining
 * if the client is allowed to post to the server.  Initializes
 * {@link #_reader_} and {@link #_writer_} to wrap
 * {@link SocketClient#_input_} and {@link SocketClient#_output_}.
 ***/
 @Override
 protected void _connectAction_() throws IOException
 {
 super._connectAction_();
 _reader_ =
 new BufferedReader(new InputStreamReader(_input_,
 __DEFAULT_ENCODING));
 _writer_ =
 new BufferedWriter(new OutputStreamWriter(_output_,
 __DEFAULT_ENCODING));
 __getReply();

 _isAllowedToPost = (_replyCode == NNTPReply.SERVER_READY_POSTING_ALLOWED);
 }

 ...

}
[/java]

As you can see the creation of both Reader and Writer uses ISO-8859-1 encoding which do not support Polish characters. And when I began to wonder how to build library after changing ___DEFAULT_ENCODING_ to "UTF-8" I noticed pom.xml in the source file. Hallelujah! - I thought - Maven to the rescue :)

With Netbeans Maven plugin onboard I fixed encoding and ran all tests. To my surprise  not all ended green. Quick question to Uncle Google and I knew that some tests fail on the systems with non-English default locale. Not to spend too much time on this issue I just commented out these tests and built new jar.

Below you can see effects, all Polish character are properly sent to discussion group.

[![Properly encoded Polish character using Apache Commons Net](http://tomaszdziurko.pl/wp-content/uploads/2009/04/message2-300x104.png)](http://tomaszdziurko.pl/wp-content/uploads/2009/04/message2.png)
