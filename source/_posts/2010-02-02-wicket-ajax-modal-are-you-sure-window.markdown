---
author: Tomasz Dziurko
comments: true
date: 2010-02-02 08:43:34+00:00
layout: post
slug: wicket-ajax-modal-are-you-sure-window
title: Wicket Ajax Modal ‘Are you sure?’ window
wordpress_id: 239
categories:
- How-to
- Java
- Wicket
tags:
- are you sure
- How-to
- Java
- modal window
- Wicket
---

While developing web application with Wicket I sometimes need to check  whether the user really, really does want to do something, for example  to delete an entity from the database. The first and easiest choice that  comes to my mind is to use JavaScript window.

So we have HomePage.html:

``` html
<!DOCTYPE html
    PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns:wicket="http://wicket.apache.org/dtds.data/wicket-xhtml1.4-strict.dtd" >
    <head>
        <title>Wicket Ajax 'Are you sure?' Modal Window</title>
    </head>
    <body>

        <div align="center">
            <strong>Wicket Ajax 'Are you sure?' Modal Window</strong>

            <form action="" wicket:id="formWithJavaScript">
                <input type="submit" wicket:id="buttonWithJavaScript" value="Action!"/>
            </form>
        </div>

    </body>
</html>
```

and corresponding Java class:

``` java
package pl.tdziurko.ajaxmodalwindowapp;

// imports omitted

public class HomePage extends WebPage {

    private static final long serialVersionUID = 1L;

    public HomePage(final PageParameters parameters) {

        Form formWithJavaScript = new Form("formWithJavaScript");

        Button buttonWithJavaScript = new Button("buttonWithJavaScript") {

            @Override
            public void onSubmit() {
                System.out.println("Doing my job");
            }
        };
        buttonWithJavaScript.add(new SimpleAttributeModifier(
                "onclick", "if(!confirm('Do you really want to perform this action?')) return false;"));

        formWithJavaScript.add(buttonWithJavaScript);
        add(formWithJavaScript);

    }

}
```

Finally, we can see how it looks:

->![](/images/blog/2010/11/javaScriptWindow.png)<-
It solves our problem but in the era of Web2.0, rounded corners and  shiny looks it isn't enough. Why can't we use ajax modal window to ask  user for confirmation? It would make our application look good and our  css magician could make it look even better.<!-- more -->
So let's try with creating reusable 'Are you sure?' ajax modal window with Wicket.

At the beginning we must prepare panel which will be displayed in our modal window. Let's name it YesNoPanel.

``` html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html
    PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns:wicket>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
        <title></title>
    </head>
    <body>
        <wicket:panel>
            <form wicket:id="yesNoForm" action="">
                <span wicket:id="message">Are you sure?</span>
                <table style="width: 65%;" align="center">
                    <tr>
                        <td align="left">
                            <input type="submit" wicket:id="noButton" value="No" />
                        </td>
                        <td align="right">
                            <input type="submit" wicket:id="yesButton" value="Yes" />
                        </td>
                    </tr>
                </table>
            </form>
        </wicket:panel>
    </body>
</html>
```

and Java class:

``` java
package pl.tdziurko.ajaxmodalwindowapp.areyousuremodal;

import org.apache.wicket.ajax.AjaxRequestTarget;
import org.apache.wicket.ajax.markup.html.form.AjaxButton;
import org.apache.wicket.extensions.ajax.markup.html.modal.ModalWindow;
import org.apache.wicket.markup.html.basic.MultiLineLabel;
import org.apache.wicket.markup.html.form.Form;
import org.apache.wicket.markup.html.panel.Panel;
import pl.tdziurko.ajaxmodalwindowapp.areyousuremodal.AreYouSurePanel.ConfirmationAnswer;

public class YesNoPanel extends Panel {

    public YesNoPanel(String id, String message, final ModalWindow modalWindow, final ConfirmationAnswer answer) {
        super(id);

        Form yesNoForm = new Form("yesNoForm");

        MultiLineLabel messageLabel = new MultiLineLabel("message", message);
        yesNoForm.add(messageLabel);
        modalWindow.setTitle("Please confirm");
        modalWindow.setInitialHeight(200);
        modalWindow.setInitialWidth(350);

        AjaxButton yesButton = new AjaxButton("yesButton", yesNoForm) {

            @Override
            protected void onSubmit(AjaxRequestTarget target, Form form) {
                if (target != null) {
                    answer.setAnswer(true);
                    modalWindow.close(target);
                }
            }
        };

        AjaxButton noButton = new AjaxButton("noButton", yesNoForm) {

            @Override
            protected void onSubmit(AjaxRequestTarget target, Form form) {
                if (target != null) {
                    answer.setAnswer(false);
                    modalWindow.close(target);
                }
            }
        };

        yesNoForm.add(yesButton);
        yesNoForm.add(noButton);

        add(yesNoForm);
    }

}
```

Everything looks pretty straightforward. We pass to the constructor text  which will be displayed as a confirmation question, references to  ModalWindow object in which YesNoPanel is placed and to  ConfirmationAnswer object.
ConfirmationAnswer class will be created  in the next paragraph and will be used to store information whether user  pressed 'Yes' or 'No' button in our panel.

Now it's time to  prepare wrapping form to our YesNoPanel. We could simply achieve it by  creating panel with form and one button in it. In our example it will be  AreYouSurePanel class:

``` html
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE html
    PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns:wicket>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
        <title></title>
    </head>
    <body>
        <wicket:panel >
            <form wicket:id="confirmForm" action="" style="display: inline;">
                <input type="submit" wicket:id="confirmButton" value="Default name" />
                <span style="display: none;" wicket:id="modal"></span>
            </form>
        </wicket:panel>
    </body>
</html>
```

and in Java:

``` java
package pl.tdziurko.ajaxmodalwindowapp.areyousuremodal;

import java.io.Serializable;
import java.util.Map;
import org.apache.wicket.ajax.AjaxRequestTarget;
import org.apache.wicket.ajax.markup.html.form.AjaxButton;
import org.apache.wicket.extensions.ajax.markup.html.modal.ModalWindow;
import org.apache.wicket.markup.html.form.Form;
import org.apache.wicket.markup.html.panel.Panel;
import org.apache.wicket.model.Model;

public abstract class AreYouSurePanel extends Panel {

    protected ModalWindow confirmModal;
    protected ConfirmationAnswer answer;
    protected Map<string,string> modifiersToApply;

    public AreYouSurePanel(String id, String buttonName, String modalMessageText) {
        super(id);
        answer = new ConfirmationAnswer(false);
        addElements(id, buttonName, modalMessageText);
    }

    protected void addElements(String id, String buttonName, String modalMessageText) {

        confirmModal = createConfirmModal(id, modalMessageText);

        Form form = new Form("confirmForm");
        add(form);

        AjaxButton confirmButton = new AjaxButton("confirmButton", new Model(buttonName)) {

            @Override
            protected void onSubmit(AjaxRequestTarget target, Form form) {
                confirmModal.show(target);
            }
        };

        form.add(confirmButton);

        form.add(confirmModal);

    }

    protected abstract void onConfirm(AjaxRequestTarget target);
    protected abstract void onCancel(AjaxRequestTarget target);

    protected ModalWindow createConfirmModal(String id, String modalMessageText) {

        ModalWindow modalWindow = new ModalWindow("modal");
        modalWindow.setCookieName(id);
        modalWindow.setContent(new YesNoPanel(modalWindow.getContentId(), modalMessageText, modalWindow, answer));
        modalWindow.setWindowClosedCallback(new ModalWindow.WindowClosedCallback() {

            @Override
            public void onClose(AjaxRequestTarget target) {
                if (answer.isAnswer()) {
                    onConfirm(target);
                } else {
                    onCancel(target);
                }
            }
        });

        return modalWindow;
    }

    public class ConfirmationAnswer implements Serializable {

        private boolean answer;

        public ConfirmationAnswer(boolean answer) {
            this.answer = answer;
        }

        public boolean isAnswer() {
            return answer;
        }

        public void setAnswer(boolean answer) {
            this.answer = answer;
        }
    }

}
```

Here we do following steps:



	
  1. Create form with one AjaxButton which shows modalWindow when clicked.

	
  2. Create modalWindow with YesNoPanel in it. As mentioned earlier, we pass  there references to our modal window and to confirmationAnswer object.

	
  3. Add WindowClosedCallback to modalWindow and basing on user choice  perform onConfirm or onCancel method. These methods are both abstract to  force developer extending AreYouSurePanel to implement them according  to his needs.


That's it, we are done. To test how it's working we must change a bit our page class and html file:

``` html
<!DOCTYPE html
    PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html xmlns:wicket="http://wicket.apache.org/dtds.data/wicket-xhtml1.4-strict.dtd" >
    <head>
        <title>Wicket Ajax 'Are you sure?' Modal Window</title>
    </head>
    <body>

        <div align="center">
            <strong>Wicket Ajax 'Are you sure?' Modal Window</strong>

            <form action="" wicket:id="formWithJavaScript">
                <input type="submit" wicket:id="buttonWithJavaScript" value="Action!"/>
            </form>
        </div>

        <div align="center">
            <span wicket:id="yesNoPanel"/>
        </div>

    </body>
</html>
```

and

``` java
package pl.tdziurko.ajaxmodalwindowapp;

// imports omitted

public class HomePage extends WebPage {

    private static final long serialVersionUID = 1L;

    public HomePage(final PageParameters parameters) {

        Form formWithJavaScript = new Form("formWithJavaScript");

        Button buttonWithJavaScript = new Button("buttonWithJavaScript") {

            @Override
            public void onSubmit() {
                System.out.println("Doing my job");
            }
        };
        buttonWithJavaScript.add(new SimpleAttributeModifier(
                "onclick", "if(!confirm('Do you really want to perform this action?')) return false;"));

        formWithJavaScript.add(buttonWithJavaScript);
        add(formWithJavaScript);

        AreYouSurePanel yesNoPanel = new AreYouSurePanel("yesNoPanel", "Ajax Action!", "Do you really want to perform this action?") {

            @Override
            protected void onConfirm(AjaxRequestTarget target) {
                System.out.println("Doing my job after ajax modal");
            }

            @Override
            protected void onCancel(AjaxRequestTarget target) { }

        };

        add(yesNoPanel);
    }

}
```

And after clicking 'Ajax Action!' we could see that it's working as intended:

->![](/images/blog/2010/11/ajaxModalWindow.png)<-
