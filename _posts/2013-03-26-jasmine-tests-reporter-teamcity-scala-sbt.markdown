---
layout: post
title: Jasmine tests reporter for TeamCity in Scala/sbt
description: Making sbt-jasmine-plugin report JavaScript tests to TeamCity
date: 2013-03-26
author: tomek
image: '/images/jasmine-teamcity-cover.jpg'
tags: [Programming, JavaScript]
tags_color: '#5fb05f'
---

This post is basically a re-post of an article I have written on the [blog of my company](http://softwaremill.com/bootzooka-iteration-11-project-rename-and-jasmine-tests-visibility-in-teamcity). But before you go there to read the full story, please check this short introduction below to decide if you are interested 🙂

In [our company](http://softwaremill.com/) sandbox project called [Bootzooka](https://github.com/softwaremill/bootzooka) we are using pretty cool technology stack: Scala, Scalatra, SBT and AngularJS. Of course we have some JavaScript tests that are executed using [sbt jasmine plugin](https://github.com/guardian/sbt-jasmine-plugin) from Guardian. The current 0.8 version of this plugin allows to execute these tests in TeamCity and even fails the build when any JS test is not passing, but it didn’t provide a full test report for these tests. This plugin worked quite well for us, but we wanted something more and that’s why we added a story saying “_As a Developer I want to see jasmine tests in TeamCity report_” to our Bootzooka backlog.

To sum up, we have added support to TeamCity [Reporting Tests](http://confluence.jetbrains.com/display/TCD65/Build+Script+Interaction+with+TeamCity#BuildScriptInteractionwithTeamCity-ReportingTests) feature allowing CI to react on specific log messages produced during different test execution phases and after that we released our forked version [0.8.1 of sbt jasmine plugin](https://github.com/softwaremill/sbt-jasmine-plugin/tree/0.8.1) and also made a pull request to the Guardian project.

The whole process of plugin modification is described in [this blog post](http://softwaremill.com/bootzooka-iteration-11-project-rename-and-jasmine-tests-visibility-in-teamcity) mentioned above.
