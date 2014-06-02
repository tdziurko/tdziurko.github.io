---
author: Tomasz Dziurko
comments: true
date: 2011-07-24 13:17:37+00:00
layout: post
slug: conversation-scope-wicket-cdi
title: Conversation scope in Wicket using CDI
wordpress_id: 804
keywords: wicket, conversation, conversation scope, CDI
categories:
- Java
- Wicket
tags:
- CDI
- conversation
- Weld
---

Some time ago I was asked to do some research about integrating Wicket with conversation scope from Context and Dependency Injection (CDI) from JEE6 specification. And as not everything was clear and easy to run, I decided to post my findings in this blog.


### Project setup


To make things easier we will base on existing example web application showing how to integrate Wicket with Weld (CDI) from project [Seam-Wicket](http://seamframework.org/Seam3/WicketModule) (formerly named [weld-wicket](http://weld-development-discussions.46994.n3.nabble.com/Move-weld-wicket-to-seam-wicket-td724181.html)). So first let's pull latest version of Seam-Wicket project from Git:

``` xml
git clone git://github.com/seam/wicket.git
cd wicket
mvn clean install
```

and in examples directory there is numberguess Maven project. Just import it into your favourite IDE and execute

``` xml
mvn install jetty:run -P jetty
```

<!-- more -->

And when you open browser you should see

    
    <a href="/images/blog/2011/07/numberguess.png"><img src="/images/blog/2011/07/numberguess.png" title="NumberGuess example application" height="199" width="814" alt="" class="aligncenter size-full wp-image-809"></img></a>


It is a simple game in which user must guess a number.


### Under the hood


And if you dive into source code there are some CDI features working under the hood:

1. Our NumberGuessApplication class extends SeamApplication which adds component instatiation listener for auto-injecting CDI beans to Wicket components and custom SeamWebRequestCycleProcessor with SeamRequestCycle alongside to manage propagation of conversation between requests.

2. Generator class is @ApplicationScoped (single instance for application) bean responsible for preparing random number to guess. We can find @Produces annotation there which marks methods responsible for creating beans of specified type. In this class those are @Random and @MaxNumber beans.

3. Game class with @SessionScoped annotation, so each user gets his own game connected with his session. Game class stores drawn number, number of guesses left, etc. The role of CDI is to inject two beans mentioned earlier, random number and maxNumber into Game object.

4. HomePage class containing form to pick a number and showing status of the game. This class has injected two interesting objects: Game bean and Conversation bean on which we will focus later in this post.

5. And of course we can't forget about empty beans.xml file under META-INF directory which is necessary for CDI to work.


### Conversation scope with Wicket


Ok, so let's first describe what conversation scope is. You can consider it as scope between request scope (everything is done in one request) and session scope. Session scope can last for unlimited time and undefined number of requests but conversation scope is set of requests coming one after another and sharing the same conversation id. The simplest use case of conversation is a wizard when user populates data in many steps with many smaller forms instead of one huge. And when user reaches last one and clicks submit, we finish conversation and send all data to the database.

If you check HomePage.java source code carefully there is conversation injected and even started there. But explicit ending command is missing so although this example project is showing how to integrate Wicket and CDI nicely, it lacks some code showing conversation scope in work. And that's why the most interesting things in this post start here.


### More conversational Wicket-CDI example


Of course my modifications of this example won't be very sophisticated. I just want to show that conversation scope is working with Wicket and (as not every use case will work out of the box) what you can do about it

Ok, so let's start with something simple. As you already know Game class which holds state of user game is @SessionScoped bean so if user opens new browser tab with NumberGuess web application, he will see state of the game he started in his first tab. That is not ok as he might want to run many games simultaneously. So we replace session annotation with @ConversationScoped and add label showing current conversation id to show what conversation is currently running.

``` java
public class HomePage extends WebPage {
    @Inject
    private Conversation conversation;
        // (...)
        public HomePage() {
        // (...)
        Label conversationLabel = new Label("conversationId", conversation.getId());
        form.add(conversationLabel);
        // (...)
    }
}
```

When you open our application in two tabs you should see something like this:

->[](/images/blog/2011/07/conversationLabel.png)<-

and in these two tabs you have two separate games running (and different conversation id shown). So everything looks fine. But, but... if during the game you accidentally hit refresh button (or F5) you will lose your state of game and see that conversation id has changed. WTH?! Calm down and check SeamApplication class source code. There is custom WebRequestCycleProcessor named SeamWebRequestCycleProcessor:

``` java
public class SeamWebRequestCycleProcessor extends WebRequestCycleProcessor {
    @Inject
    Conversation conversation;

    /**
     * If a long running conversation has been started, store its id into page
     * metadata
     */
    @Override
    public void respond(RequestCycle requestCycle) {
        super.respond(requestCycle);
        if (!conversation.isTransient()) {
            Page page = RequestCycle.get().getResponsePage();
            if (page != null) {
                page.setMetaData(SeamMetaData.CID, conversation.getId());
            }
        }

    }
}
```

And what is most important here is that conversation id is saved between requests. I repeat, **requests**. And if you check what type GuessButton in HomePage.java is, you will see AjaxButton so there is no new HTTP request fired when it is clicked. And as a result no information about conversation is saved. Such behavior can be considered as limitation but in most cases you won't need conversation scope until you change page, so HTTP request is used and problem with missing information about conversation id disappears.

Ok, so if I change button to non-ajax, everything will work as expected. I also changed "restart button" to end conversation and redirect to HomePage to reset the game automatically. And when we try play the game, we will see that guess button is generating full page refresh and hitting F5 doesn't cause problems.


### Conversation between different pages


So far so good. But how conversation scope is working when we try to redirect to another page? Let's find out. First we should create SecondPage java and html files:

``` java
public class SecondPage extends WebPage {

    @Inject
    private Game game;

    @Inject
    private Conversation conversation;

    public SecondPage() {
        Label conversationLabel = new Label("conversationId", conversation.getId());
        add(conversationLabel);

        Label guessesLeft = new Label("guessesLeft", game.getRemainingGuesses() + "");
        add(guessesLeft);
    }
}
```



``` html

<h1>Second page</h1>
<div>Conversation id:</div>
<div>Guesses left:</div>

```

with information about conversation id and number of remaining guesses. Because Wicket propagates conversation between requests, we should get the same conversation id and the same game instance injected into both pages sharing the same conversation. But to test it we need a link from HomePage to SecondPage, so let's add a simple one:

``` java
Link secondPageLink = new Link("secondPageLink") {
    @Override
    public void onClick() {
        setResponsePage(new SecondPage());
    }
};
```

with corresponding HTML element in HomePage.html file.

And when we try to guess a number and then click new link, we will see:


->![](/images/blog/2011/07/withLink.png)<-

and then

->![](/images/blog/2011/07/secondPage.png)<-

So as you can see, we have the same conversation id and the same game instance. So it's working.

**Bookmarkable pages**
Now we will try with link not pointing to the specific page instance (created in onClick() method) but to SecondPage class in general, so when user clicks, Wicket creates page of class passed to setResponsePage() method. This kind of link redirects to bookmarkable page because it doesn't hold a state. Application knows what to render basing only on information stored in the url.

``` java
Link secondPageLink = new Link("secondPageLink") {
    @Override
    public void onClick() {
        setResponsePage(SecondPage.class);
    }
};
```

And when we click this link, everything seems to be the same. We see the same page with correct conversation id, etc., but one thing has changed, the url. Now it looks like that

``` xml
http://localhost:9090/wicket-numberguess/?wicket:bookmarkablePage=:
    org.jboss.seam.wicket.examples.numberguess.SecondPage&cid=1
```

The most interesting part is the last parameter "cid" (conversationId) which allows Wicket to inject proper conversation into newly created page. The reason why we see this parameter is that our url points to bookmarkable page, so application must be able to create complete and configured page basing only on data stored in this url. Of course our conversation can expire before someone clicks this link so we should be prepared to recover from such situation (for example by simply overriding handleMissingConversation from SeamRequestCycle class).

Ok, so default bookmarkable pages are supported. But as you probably noticed, our last url contains package name so it is far from being elegant and SEO friendly. That's why we will try to mount it with some short, pretty url name "second-page". To achieve it, we must add one line to init method in our NumberGuessApplication class:

``` java
    @Override
    protected void init() {
        super.init();
        mountBookmarkablePage("second-page", SecondPage.class);
    }
```

and when we click the same link we will land on page with address

``` xml
http://localhost:9090/wicket-numberguess/second-page/cid/1
```

And we could expect that the same conversation is active. But unfortunately it's not. Custom SeamRequestCycle is not able to automatically extract conversation id from url unless it is specified in a standard way as ?cid=X . To make sure that this is the problem I've done some tests and different url coding strategies (classes extending [AbstractRequestTargetUrlCodingStrategy](http://wicket.apache.org/apidocs/1.4/index.html?org/apache/wicket/request/target/coding/AbstractRequestTargetUrlCodingStrategy.html) ) are working or not depending on the fact how they append parameters. Those appending them as /paramName/paramValue are failing but those appending parameters as ?paramName=paramValue are working fine. Ok, you may think that we are stuck now. But remember, this is SPARTA!... uhm.. open source so if we need different behavior we could write it.


### Custom SeamRequestCycle implementation


All we need is to look into source code of SeamRequestCycle and make some small improvements

``` java
public class SeamRequestCycle extends WebRequestCycle {
    @Override
    protected void onRequestTargetSet(IRequestTarget target) {
        super.onRequestTargetSet(target);

        Page page = null;
        if (target instanceof IPageRequestTarget) {
            page = ((IPageRequestTarget) target).getPage();
        }

        // Two possible specifications of cid: page metadata or request url; the
        // latter is used to propagate the conversation to mounted (bookmarkable)
        // paths after a redirect

        String cid = null;
        if (page != null) {
            cid = page.getMetaData(SeamMetaData.CID);
        } else {
            cid = request.getParameter("cid");
        }

        ConversationContext conversationContext = instance().select(HttpConversationContext.class).get();
        if (!conversationContext.isActive())
            conversationContext.activate(cid);
        Conversation conversation = conversationContext.getCurrentConversation();

        // handle propagation of existing long running converstaions to new
        // targets
        if (!conversation.isTransient()) {
            // Note that we can't propagate conversations with other redirect
            // targets like RequestRedirectTarget through this mechanism, because
            // it does not provide an interface to modify its target URL. If
            // propagation with those targets is to be supported, it needs a custom
            // Response subclass.
            if (isRedirect() && target instanceof BookmarkablePageRequestTarget) {
                BookmarkablePageRequestTarget bookmark = (BookmarkablePageRequestTarget) target;
                // if a cid has already been specified, don't override it
                if (!bookmark.getPageParameters().containsKey("cid"))
                    bookmark.getPageParameters().add("cid", conversation.getId());
            }

            // If we have a target page, propagate the conversation to the page's
            // metadata
            if (page != null) {
                page.setMetaData(SeamMetaData.CID, conversation.getId());
            }
        } else if (cid != null) {
            handleMissingConversation(cid);
        }

    }

    // (...)
}
```

So what we can see here is:
1. Getting cid from request parameters or from page metadata.
2. Activation of found conversation.
3. Copying conversation id to PageParameter or to Page MetaData which allows later injection of correct conversation.
4. Execution of handleMissingConversation method if found conversation is expired.

And most critical part of code which needs small tuning is:

``` java
String cid = null;
if (page != null) {
    cid = page.getMetaData(SeamMetaData.CID);
} else {
     cid = request.getParameter("cid");
}
```

We just must find the way how to extract cid parameter from the url. After some research I found that request.getURL() method returns part of address which contains cid parameter name and value. So simple regex magic and here we are:

``` java
    String cid = null;
    if (page != null) {
        cid = page.getMetaData(SeamMetaData.CID);
    } else {
        cid = request.getParameter("cid");
    }

    if (cid == null) {
        Pattern pattern = Pattern.compile("/cid/(\\d+)($|/)");
        Matcher matcher = pattern.matcher(request.getURL());

        if (matcher.find()) {
            cid = matcher.group(1);
            System.out.println("cid = " + cid);
        }
    }
```

When none of standard ways of extracting cid work, application tries to extract it from url. Of course in more complicated project this pattern might need some additional tweaks but for us it is enough.

And to use our modification we copy content of SeamRequestCycle into our CustomSeamRequestCycle class, apply our small patch and then configure NumberGuessApplication to use our implementation:

``` java
    @Override
    public RequestCycle newRequestCycle(final Request request, final Response response) {
        return new CustomSeamRequestCycle(this, (WebRequest) request, (WebResponse) response);
    }
```

And when we restart jetty and use link to the SecondPage, we will see correct conversation and game object injected! :)


### Summary


As you can see conversation scope from CDI (Weld) in most cases is working with Wicket without any problems. For some specific use cases it needs some tweaks but everything can be achieved easily. So if you are planning to start new Wicket project in the near future and you want to learn some new and interesting technology, combining your beloved web framework with Context and Dependency Injection (which is very similar to Spring 3) from JEE6 might be a good idea.

All source code from this post is available in [my BitBucket repo](https://bitbucket.org/tdziurko/wicket-cdi-extended).
