---
author: Tomasz Dziurko
comments: true
date: 2012-05-13 06:48:54+00:00
layout: post
slug: jcommander-parsing-command-line-parameters-ease
title: JCommander - parsing command line parameters with ease
wordpress_id: 1426
categories:
- Java
tags:
- Cedric Beust
- jcommander
- parsing command line
---

From time to time each of us have to create a small console application to perform a few tedious tasks that can be automated so we can concentrate on more creative ones instead. And every time I had to build such an app I struggled with passing options via command line parameters. Instead of concentrating on implementing logic, I had to create a parser, validator and so on. And in another console application I had to do the same... again and again. But thankfully it's over. 

Thanks to small library by [Cédric Beust](http://beust.com/), creator of well-known Java testing framework [TestNG](http://testng.org/). This time he created **JCommander** with a motto _Because life is too short to parse command line parameters_. But enough talking, let's sit down and write some code.
<!-- more -->


### A simple use case


At the begginning assume that we have to write an application to generate a timesheet report from a given month. So simplified use case is: connect to application where all data is stored and load what everything we need. To perform this operation we need four parameters: url to the application where data is stored, authentication token and of course number of month and year. So let's create a settings class where all this data will be stored.

``` java
public class Settings {

    private String url;

    private String authenticationToken;

    private Integer month;

    private Integer year;

}
```

and our Main application class with _main()_ method:

``` java
public class Main {

    public static void main(String[] args) {

        Settings settings = new Settings();
        // somehow pass args[] to settings object

        // do the logic
    }
}
```

But how should we pass all stuff from command line parameters to the _Settings_ class so it is automatically initialized with these values? It's simple!

First we have to annotate fields in _Settings_ class using _@Parameter_ so JCommander will know how to map parameters from command line to fields.

``` java
public class Settings {

    @Parameter(names = "-url", description = "Server address", required = true)
    private String url;

    @Parameter(names = "-token", description = "Authentication token", required = true)
    private String authenticationToken;

    @Parameter(names = "-month", description = "Number of month (1-12) for timesheet", required = true)
    private Integer month;

    @Parameter(names = "-year", description = "Year", required = true)
    private Integer year;

}
```

Annotation we've added looks pretty straightforward. We define name of parameter and whether it is required or not. If we do not provide a token (which is required) in command line we will get something similar to:

    
    Exception in thread "main" com.beust.jcommander.ParameterException: The following options are required: -token


And in next step we have to inform JCommander which object should be initialized with data extracted from command line. It's a simple one-liner.

``` java
public class Main {

    public static void main(String[] args) {

        Settings settings = new Settings();
        new JCommander(settings, args); // simple one-liner
        // do the logic
    }
}
```

And that's all. We have simple JCommander example working and all settings passed via command line params are now initialized in Settings object.


### Some additional options


**Multiple values**

But then we receive some new feature requests. We have to generate report for a list of projects. It can be only one project but it can be more. So how can we implement it? Nothing difficult. We add _List<String>_ projects to our settings class and mark it as a required parameter.

``` java

    @Parameter(names = "-project", 
        description = "Codes of projects to be included in timesheet report", required = true)
    private List<String> projectCodes;

```
Seeing a Collection JCommander will detect that it should expect multiple values for project parameter name. And if we call our console application with something like

    
    java -jar Application.jar -project Project1 -project Project2 - project Project3


we will get list of these names in _projectCodes_ field.

**Input validation**

If we want to validate that month parameter is between 1 and 12 we can do it easily with just one method from interface _IParameterValidator_. In our case it will look like show below:

``` java
public class MonthValidator implements IParameterValidator {

    @Override
    public void validate(String name, String value) throws ParameterException {
        int month = Integer.parseInt(value);

        if(month < 1 || month > 12) {
            throw new ParameterException("Parameter " + name + " should be between 1 and 12");
        }
    }
}
```

and we have to declare this validator using _validateWith_ from _@Parameter_ annotation:

``` java
    @Parameter(names = "-month", 
        description = "Number of month (1-12) for timesheet", required = true, validateWith = MonthValidator.class)
    private Integer month;
```

**Input conversion**

Sometimes we need to have one of our options mapped to something more than a simple basic type. Let's say that in our example console application user should provide type of report output he/she wants to get. It could be console output, pdf or xls file. So in out _Settings_ class we have a enum _OutputEnum_ with three allowed values:

``` java
public class Settings {

    // ...
    @Parameter(names = "-output", description = "Report output format", required = true)    
    private OutputEnum output;

}

public enum OutputEnum {

    CONSOLE,
    PDF,
    XLS;

    // converter that will be used later
    public static OutputEnum fromString(String code) {

        for(OutputEnum output : OutputEnum.values()) {
            if(output.toString().equalsIgnoreCase(code)) {
                return output;
            }
        }

        return null;
    }
}

```

But now JCommander doesn't know how to work with our new enum class. We need to provide a converter. So let's create one. And again, it's easy :) We have to implement one method from _IStringConverter_ and throw _ParameterException_ when something is wrong.

``` java
public class OutputConverter implements IStringConverter<OutputEnum> {

    @Override
    public OutputEnum convert(String value) {
        OutputEnum convertedValue = OutputEnum.fromString(value);

        if(convertedValue == null) {
            throw new ParameterException("Value " + value + "can not be converted to OutputEnum. " +
                    "Available values are: console, pdf, xls.");
        }
        return convertedValue;
    }
}

```

and when converter is ready, we should inform JCommander to use it on one of fields in Settings class with _converter_ annotation parameter.

``` java
public class Settings {

    // ...
    @Parameter(names = "-output", description = "Report output format", required = true, , converter = OutputConverter.class)    
    private OutputEnum output;

}
```

And now settings object will be correctly initialized with OutputEnum.


### Summary


That's all. I've shown some most useful features of JCommander, an easy tool to make parsing command line paramters a breeze. If you want some more info, please check official [JCommander website](http://jcommander.org) or even dive into its [source code](https://github.com/cbeust/jcommander) on GitHub.
