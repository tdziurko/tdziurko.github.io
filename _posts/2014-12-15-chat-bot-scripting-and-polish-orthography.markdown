---
layout: post
title: Chat bot scripting and Polish orthography
description: A Hubot script for Slack gently correcting common Polish spelling mistakes
date: 2014-12-15
author: tomek
image: '/images/chat-bot-slap.jpg'
tags: [Programming, JavaScript]
tags_color: '#5fb05f'
---

A few lines of script to help developers to learn orthography.

In [our remote company](http://softwaremill.com/) we are heavily using text messaging tools, we’ve tried Skype, then HipChat and then [Slack](https://slack.com/) which we are using until now. One of the coolest features in Slack is its scripting capability. Moreover Github built [Hubot](https://hubot.github.com/) – a bot running our scripts. It can be easily integrated with Slack. You deploy the bot on Heroku, put custom scripts in the proper directory and that’s it.

## The Idea

We have been using Slack for some time and I’ve planned to play with custom scripting since the beginning of Slack in SoftwareMill. Yet I didn’t have any idea good enough that I would love to implement it right away.
That was the case until last week when I’ve noticed that we are making quite a lot of orthographic mistakes. And when you often see words written with an error, your internal ‘correct or not’ detector starts to go haywire – you are no longer sure if that word should be written this way or another. So I’ve decided to write a script that will try to spot such errors, correct and comment on them in a gentle way 🙂

![](/images/chat-bot-slap.jpg)

### Intro

But first let me briefly describe the anatomy of a Hubot script. It can be written in JavaScript or CoffeeScript and should implement at least one of two methods:

```coffeescript
module.exports = (robot) ->
robot.hear <regexp>, (msg) ->
# Code reacting on any message written in any chat room that matches <regexp>
msg.send 'Example response to chat room'

robot.respond <regexp>, (msg) ->
# Code reacting on any message written to our bot that matches <regexp>
msg.send 'Example response 2 to chat room'
```

Because we are interested in watching every room (channel) we will only play with _hear_ method.

### Iteration 1

At the beginning I wanted to implement something simple to check that my idea is working and to see if it is easy to write and run a custom script on Hubot. And after that, I could make my script more sophisticated, etc. So at first I’ve created something like this:

```coffeescript
module.exports = (robot) ->
  robot.hear prepareErrorDetectingRegEx(), (msg) ->
    author = msg.message.user.name
    grammarFailure = msg.match[1]
    exclamationSentence = msg.random messages
    msg.send  '@' + author + ', ' + exclamationSentence + '! It should be *' + errors[grammarFailure.toLowerCase().trim()] + '*'

prepareErrorDetectingRegEx = () ->
  errorWords = []
  for k, v of errors
    errorWords.push k
  joinedErrors = errorWords.join('|')
  errorDetectingRegex = new RegExp('.*(' + joinedErrors + ').*', 'i');
  return errorDetectingRegex

# key-value pair where key is an error and value contains correct word
errors =
  'wziąść'  : 'wziąć'
  'wziasc'  : 'wziąć'
  'wziaśc'  : 'wziąć'
  'wziasć'  : 'wziąć'
  'pokarze' : 'pokażę'
  'pokarzę' : 'pokażę'
  'żądzić'  : 'rządzić'
  'żadzić'  : 'rządzić'
  'żadzic'  : 'rządzić'
  'ządzic'  : 'rządzić'

messages = ['come on', 'gimme a break', 'are you serious?'] # for a Polish list of 'gentle' messages check source code (link at the end of the post) :-)
```

The entry point of our script is in _robot.hear_ function. We are creating regEx in _prepareErrorDetectingRegEx_ which loads all _errors_ and produces _OR_ combo with all possible mistakes. This regular expression seemed fine at first, but then turned out to be not so perfect 🙂

When any text written in chat matches our pattern, we know that we have our “victim”. Now we can extract the author name, find the correct version of the word and prepare complete response like

```text
@user, gimme a break! It should be <correctVersionOfWordWrittenInBold>
```

### Iteration 2

After the first release, it was time to test the script on production and see how it deals with messages posted during normal work day. And of course, it turned out that many common errors are missing in _errors_ object but also that the first version of the regular expression is far from perfect. It was detecting mistakes in properly written words, for example if we had pair ‘eror’: ‘error’ it was firing also for messages like ‘abcEror’ or ‘erorSomething’.

This one can be fixed by a new, better pattern checking only for separate words (or start/end of sentence)

```coffeescript
prepareErrorDetectingRegEx = ->
  errorWords = []
  for k, v of errors
    errorWords.push k

  joinedErrors = errorWords.join('|')
  new RegExp '(^|\\s)(' + joinedErrors + ')($|\\s)', 'i'
```

That was the first issue, the second one was rather on the code level: in Polish we have these national characters like “ąęłśćżźóń” but some people write words without them. And to catch all possible combinations, each word had to be added to _errors_ in many versions. Below you can see example listing with only three errors but using all combinations requires 10 records:

```coffeescript
errors =
  'wziąść'  : 'wziąć'
  'wziasc'  : 'wziąć'
  'wziaśc'  : 'wziąć'
  'wziasć'  : 'wziąć'
  'pokarze' : 'pokażę'
  'pokarzę' : 'pokażę'
  'żądzić'  : 'rządzić'
  'żadzić'  : 'rządzić'
  'żadzic'  : 'rządzić'
  'ządzic'  : 'rządzić'
```

This approach is tedious and also much more error-prone. To make adding more records easier, I had to simplify the algorithm a bit. And when I was discussing this with [Szimano](http://szimano.org/) he suggested escaping all Polish characters first and after that applying regEx to detect errors. This approach was very easy to implement: first we accept all messages using _/.*/_ pattern, then replace all national characters with their standard versions:

```coffeescript
replacePolishChars = (text) ->
    text.toLowerCase()
      .replace('ą', 'a')
      .replace('ć', 'c')
      .replace('ę', 'e')
      .replace('ł', 'l')
      .replace('ń', 'n')
      .replace('ó', 'o')
      .replace('ś', 's')
      .replace('ż', 'z')
      .replace('ź', 'z')
```

Now we don’t have all those different combinations of one error word and our example errors list looks much simpler:

```coffeescript
errors =
  'wziasc'  : 'wziąć'
  'pokarze' : 'pokażę'
  'zadzic'  : 'rządzić'
```

### Backlog

So after two iterations we have a stable script doing what we want and keeping our orthography skills sharp. Complete source code is available [at GitHub](https://github.com/softwaremill/hal-9k/blob/master/scripts/grammar-corrector.coffee). But as we are using this script, some new ideas started appearing and landed in my backlog for future versions:

- Store most often misspelled words in a database
- Store users making most mistakes and print ‘hall of fame’ table
- Allow user to add new entries to _errors_ directly from chat rooms

### Summary

As you can see, scripting in text communicators like Slack or HipChat is a really powerful and easy to deploy tool, you are only limited by your creativity. And you don’t need hundreds of lines of code to write something useful.

Further reading: [Hubot Scripting Guide](https://github.com/github/hubot/blob/master/docs/scripting.md)
