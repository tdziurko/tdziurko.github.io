---
author: Tomasz Dziurko
comments: true
date: 2011-06-20 19:52:06+00:00
layout: post
slug: wicket-tutorial-part-8-%e2%80%93-adding-internationalization-support
title: Wicket Tutorial, part 8 – adding internationalization support
wordpress_id: 734
categories:
- How-to
- Item Directory application
- Java
- Wicket
- Wicket Tutorial
tags:
- i18
- internationalization
- localization
- properties
- Tutorial
---

Today we will continue Wicket tutorial series with adding support to multiple languages in web application. Ok, let's do it step by step.


### Create form to change language


To make our changes visible and testable at the beginning we add form to change application language. To make it more reusable we will implement it as a Wicket panel.
So first, let's create html and Java files:

<!-- more -->

LanguagePanel.html

[html]
    <form wicket:id="languageForm" action="">
            <input wicket:id="polishButton" type="image" src="images/pl.png"
                   name="image" width="25" height="25" style="border-style: none;"/>
            <input wicket:id="englishButton" type="image" src="images/gb.png"
                   name="image" width="25" height="25" style="border-style: none;"/>
    </form>
[/html]

LanguagePanel.java:

[java]
public class LanguagePanel extends Panel {

    Logger log = LoggerFactory.getLogger(LanguagePanel.class);

    public LanguagePanel(String id) {
        super(id);

        Form languageForm = new Form("languageForm");   // (1)

        languageForm.add(createLocaleChangingButton("polishButton", "PL"));
        languageForm.add(createLocaleChangingButton("englishButton", "GB"));

        add(languageForm);
    }

    private Button createLocaleChangingButton(final String buttonId, final String localeString) {
        return new Button(buttonId) {
            @Override
            public void onSubmit() {
                changeUserLocaleTo(localeString);    // (2)
            }
        };
    }

    private void changeUserLocaleTo(String localeString) {
        getSession().setLocale(new Locale(localeString));      // (3)
    }
}
[/java]

As you can see it's simple panel with form containing two buttons to change language to Polish and English. We just create form **(1)**, add two buttons which both call **(2)** the same method _changeUserLocaleTo() _**(3)** with different parameter. Method does one, but very important thing: it changes locale of user Sesssion object so all localized strings will be rendered in the chosen language.

Next thing we need to do is add just created panel somewhere to the application. I will place it in the header bar in the BasePage with some styling:

[html]
(...)
<ul>
    <!-- MENU -->
	<li><a>Home</a></li>
	<li><a>Item locations</a></li>
	<li><a href="#">About application</a></li>
	<li style="float: right; padding-top: 20px; padding-right: 20px;">
            <span wicket:id="languagePanel"/>
        </li>
    <!-- END MENU -->
</ul>
(...)
[/html]

We are almost done with this point but as we would like to test if language switching is working on the main page, we must add properties file for second language. And as properties gathered in Application.properties file doesn't seem to work when language is changed, we will move them to BasePage.properties:

[xml]
applicationName=Item Directory
applicationHeader=Manage your items easily!
[/xml]

and BasePage_pl.properties (for Polish language)

[xml]
applicationName=Item Directory
applicationHeader=Zarządzaj swoimi przedmiotami z łatwością!
[/xml]

Ok, it should work and when you run jetty server _mvn jetty:run_, enter application and click flag of Poland you should see:


[![](/images/blog/2011/06/item11.png)](/images/blog/2011/06/item11.png)


Small improvement can be done with titles of flag images to inform user to which locale he will switch after clicking. This is good occasion to show nice feature of wicket:message which can render not only localized text but also to localize elements in other html tags like title in input HTML element. To make it work we simply add _wicket:message="element_name:key_from_properties_bundle"_. In LanguagePanel.html we want to localize title so we add _wicket:message="title:pl"_ to input:

[html]
<form wicket:id="languageForm" action="">
    <input wicket:id="polishButton" type="image" wicket:message="title:pl" src="images/pl.png"
         name="image" width="25" height="25" style="border-style: none;"/>
     <input wicket:id="englishButton" type="image" wicket:message="title:gb" src="images/gb.png"
         name="image" width="25" height="25" style="border-style: none;"/>
</form>
[/html]

and after adding pl and gb keys to properties file we could see localized "on hover" title messages over both flags.


### Localizing messages shown to the user


After doing some tedious job (not covered here, but visible in commits) with replacing static texts with wicket:message elements and completing properties files with new keys we could introduce another Wicket feature when it comes to localization: localizing text added or created inside the Java code.

First example of using localization in Wicket inside Java files is welcome text on the main page. There are two ways to localize it. The worse one is getString(propertyName) method which returns localized String. But this method has one big drawback, it will work only when new instance of Page is created. When you (like in our language switching form) return to the current page, it will use old text value in previously selected language. This drawback is not present in the second way with [StringResourceModel](http://wicket.apache.org/apidocs/1.4/org/apache/wicket/model/StringResourceModel.html) class:

HomePage.java:

[java]
	private void initGui() {
		add(new Label("helloLabel", new StringResourceModel("welcomeLabel", this, null)));
		add(new Label("currentTime", new Date().toString()));

		add(new Label("numberOfUsers", userService.size() +""));
	}
[/java]

If you are more interested in arguments of this class constructor please read [javadocs](http://wicket.apache.org/apidocs/1.4/org/apache/wicket/model/StringResourceModel.html) where you can find detailed description how all this stuff is working.

Another place we could use StringResourceModel is feedback message after location was deleted:

RemoveLocationLink fragment:

[java]
	@Override
	public void onClick() {
		locationService.remove(location);
        getSession().info(new StringResourceModel("locationRemoved", this, null,
            new Object[] { location.getName() }).getString());
		setResponsePage(LocationsPage.class);
	}
[/java]

alongside with properties message:

[xml]
locationRemoved=Location {0} was removed
[/xml]

In the constructor of StringResourceModel we pass array of parameters which is used to prepare message. You can notice simple way of inserting passed values into text with {0} notation (like in [MessageFormat](http://download.oracle.com/javase/1.4.2/docs/api/java/text/MessageFormat.html) class). Of course this and other constructors of SRM are explained well in javadoc mentioned above :) Also worth to notice is call of getString() method on StringResourceModel which returns String representation of this ResourceModel object.


### **Localizing complete pages or panels**


Sometimes adding wicket:message isn't enough and we might want to localize whole page or panel. The first example which comes to my mind is page with regulations in e-shop web application. There would be large amount of text which should be translated to language chosen by the user. Of course we could use many, many wicket:message elements but there is better and faster way :)


To show how it works we will create localized sidebar panel of our application:




[![](/images/blog/2011/06/item2.png)](/images/blog/2011/06/item2.png)


First we create empty Java class for panel and move sidebar markup to corresponding panel HTML file:

[java]
public class RightSidebarPanel extends Panel {

    public RightSidebarPanel(String id) {
        super(id);
    }
}
[/java]
[html]

<!-- SIDEBAR -->
<h4>About</h4>
This is example application to help you manage your items hidden in many locations across your home.
<h4>Links</h4>
<ul class="blocklist">
  <li>
    <a title="Source code of this application" href="https://bitbucket.org/tdziurko/item-directory/src">
      Source code of this application
    </a>
  </li>
  <li>
    <a title="Tutorial about creating this application"
      href="<a href="/2011/03/wicket-tutorial-series-building-web-application-scratch/">
      Tutorial about creating this application
    </a>
  </li>
  <li>
    <a title="Tomasz Dziurko" href="http://tomaszdziurko.pl/">Author</a>
  </li>
  <li>
    <a title="Spyka Webmaster resources" href="http://www.spyka.net">
      Template Created by Spyka
    </a>
  </li>
</ul>
<!-- SIDEBAR -->

[/html]

We need to add just created panel to BasePage java class and then, if we run application, home page screen won't change at all. Ok, now we are ready to the final trick :) Instead of replacing static text with wicket:message elements we create RightSidebarPanel_pl.html file next to standard one and replace whole content with text in Polish language. After this, Wicket uses Polish HTML file when user choose this language and English for users visiting us from other coutries. Easy and straightforward.

Ok, that's all about localization in Wicket and in ItemDirectory application. As an extra one small improvement of usability :)


### Tweak page header and application title


One thing I was missing in previous versions of Item Directory was title linked to main page. As in many other web pages I was clicking it to navigate to home page but previously I had forgott to add this functionality to it didn't work. And now, as we are adding title in other language, it's good occasion to fix it. So first we embed title and header in a href elements in HTML page and then we add links to BasePage Java class:

[html]
  <!-- TITLE -->
  <h1><a href="#" wicket:id="headerHomePageLink"><wicket:message key="applicationName"/></a></h1>
  <h2><a href="#" wicket:id="headerHomePageLink2"><wicket:message key="applicationHeader"/></a></h2>
  <!-- END TITLE -->
[/html]
[java]
    private void addHeaderLinks() {
        add(new BookmarkablePageLink("headerHomePageLink", Application.get().getHomePage()));
	    add(new BookmarkablePageLink("headerHomePageLink2", Application.get().getHomePage()));
    }
[/java]

As you can see in HTML snippet I used _wicket:message_ element which provides simple and easy to use label in markup functionality without any Java code. So it's good to utilize this Wicket feature in places with static text which maybe will need internationalization in the future and isn't connected with any particular logic.

Thank you for reading. If you have any comments or ideas, don't hesitate to share them with me and other reader below this post.
