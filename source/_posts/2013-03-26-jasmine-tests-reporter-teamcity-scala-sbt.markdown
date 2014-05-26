---
author: Tomasz Dziurko
comments: true
date: 2013-03-26 22:10:57+00:00
layout: post
slug: jasmine-tests-reporter-teamcity-scala-sbt
title: Jasmine tests reporter in TeamCity with Scala and SBT
wordpress_id: 1775
categories:
- How-to
- JavaScript
- Scala
tags:
- Jasmine
- javascript
- sbt
- teamcity
---

<blockquote>This post is basically re-post of article I have written on my company blog [here](http://softwaremill.com/bootzooka-iteration-11-project-rename-and-jasmine-tests-visibility-in-teamcity). But before you go there to read the full story, please check this short introduction below to decide if you are interested :)</blockquote>

In [our company](http://SoftwareMill.com) sandbox project called [Bootzoooka](https://github.com/softwaremill/bootzooka) we are using pretty cool technology stack: Scala, Scalatra, SBT and AngularJS. Of course we have some JavaScript tests that are executed using [sbt jasmine plugin](https://github.com/guardian/sbt-jasmine-plugin) from Guardian. Current 0.8 version of this plugin allows to execute these tests in TeamCity and even fails build when any JS test is not passing, but it didn't provide full test report for these tests. This plugin worked quite well for us, but we wanted something more and that's why added story saying "_As a Developer I want to see jasmine tests in TeamCity report_" to our Bootzooka backlog.

To sum up, we have added support to TeamCity [Reporting Tests](http://confluence.jetbrains.com/display/TCD65/Build+Script+Interaction+with+TeamCity#BuildScriptInteractionwithTeamCity-ReportingTests) feature allowing CI to react on specific log messages produced during different test execution phases and after that we released our forked version [0.8.1 of sbt jasmine plugin](https://github.com/softwaremill/sbt-jasmine-plugin/tree/0.8.1) and also made a pull request to Guardian project.

The whole process of plugin modification is described in [this blog post ](http://softwaremill.com/bootzooka-iteration-11-project-rename-and-jasmine-tests-visibility-in-teamcity)mentioned above.
