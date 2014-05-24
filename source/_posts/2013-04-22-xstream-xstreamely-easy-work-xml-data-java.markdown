---
author: Tomasz Dziurko
comments: true
date: 2013-04-22 08:55:55+00:00
layout: post
slug: xstream-xstreamely-easy-work-xml-data-java
title: XStream - XStreamely easy way to work with XML data in Java
wordpress_id: 1784
categories:
- How-to
- Java
tags:
- Java
- parsing
- xml
- XStream
---

From time to time there is a moment when we have to deal with XML data. And most of the time it is not the happiest day in our life. There is even a term "XML hell" describing situation when programmer has to deal with many XML configuration files that are hard to comprehend. But, like it or not, sometimes we have no choice, mostly because specification from client says something like "use configuration written in XML file" or something similar. And in such cases, [XStream](http://xstream.codehaus.org/) comes with its very cool features that make dealing with XML really less painful.


[![xstream](http://tomaszdziurko.pl/wp-content/uploads/2013/04/xstream.gif)](http://tomaszdziurko.pl/wp-content/uploads/2013/04/xstream.gif)





### Overview


[XStream](http://xstream.codehaus.org/) is a small library to serialize data between Java objects and XML. It's lightweight, small, has nice API and what is most important, it works with and without custom annotations that we might be not allowed to add when we are not the owner of Java classes.
<!-- more -->


### First example


Suppose we have a requirement to load configuration from xml file:
[xml]
<config>
    <inputFile>/Users/tomek/work/mystuff/input.csv</inputFile>
    <truststoreFile>/Users/tomek/work/mystuff/truststore.ts</truststoreFile>
    <keystoreFile>/Users/tomek/work/mystuff/CN-user.jks</keystoreFile>

    <!-- ssl stores passwords-->
    <truststorePassword>password</truststorePassword>
    <keystorePassword>password</keystorePassword>

    <!-- user credentials -->
    <user>user</user>
    <password>secret</password>
</config>
[/xml]

And we want to load it into Configuration object:
[java]
public class Configuration {

    private String inputFile;
    private String user;
    private String password;

    private String truststoreFile;
    private String keystoreFile;
    private String keystorePassword;
    private String truststorePassword;

    // getters, setters, etc.
}
[/java]
So basically what we have to do is:
[java]
    
    FileReader fileReader = new FileReader("config.xml");  // load our xml file  
    XStream xstream = new XStream();     // init XStream
    // define root alias so XStream knows which element and which class are equivalent
    xstream.alias("config", Configuration.class);    
    Configuration loadedConfig = (Configuration) xstream.fromXML(fileReader);

[/java]

And that's all, easy peasy :)



### Something more serious


Ok, but previous example is very basic so now let's do something more complicated: real XML returned by real WebService.

[xml]
<DATA xmlns="">
    <BAN>
        <UPDATED_AT>2013-03-09</UPDATED_AT>
        <TROUBLEMAKER>
            <NAME1>JOHN</NAME1>
            <NAME2>EXAMPLE</NAME2>
            <AGE>24</AGE>
            <NUMBER>ASD123123</NUMBER>
        </TROUBLEMAKER>
    </BAN>
    <BAN>
        <UPDATED_AT>2012-03-10</UPDATED_AT>
        <TROUBLEMAKER>
            <NAME1>ANNA</NAME1>
            <NAME2>BAKER</NAME2>
            <AGE>26</AGE>
            <NUMBER>AXN567890</NUMBER>
        </TROUBLEMAKER>
    </BAN>
    <BAN>
        <UPDATED_AT>2010-12-05</UPDATED_AT>
        <TROUBLEMAKER>
            <NAME1>TOM</NAME1>
            <NAME2>MEADOW</NAME2>
            <NUMBER>SGH08945</NUMBER>
            <AGE>48</AGE>
        </TROUBLEMAKER>
    </BAN>
</DATA>
[/xml]

What we have here is simple list of bans written in XML. We want to load it into collection of Ban objects. So let's prepare some classes (getters/setters/toString omitted):
[java]
public class Data {
    private List bans = new ArrayList(); 
}

public class Ban {
    private String dateOfUpdate;
    private Person person;
}

public class Person {
    private String firstName;
    private String lastName;
    private int age;
    private String documentNumber;
}
[/java]

As you can see there is some naming and type mismatch between XML and Java classes (e.g. field name1->firstName, dateOfUpdate is String not a Date), but it's here for some example purposes.

So the goal here is to parse XML and get Data object with populated collection of Ban instances containing correct data. Let's see how it can be achieved.



#### Parse with annotations


First, easier way is to use annotations. And that's the suggested approach in situation when we can modify Java classes to which XML will be mapped. 
So we have:
[java]
@XStreamAlias("DATA") // maps DATA element in XML to this class
public class Data {

    // Here is something more complicated. If we have list of elements that are 
    // not wrapped in a element representing a list (like we have in our XML: 
    // multiple <ban> elements not wrapped inside <bans> collection, 
    // we have to declare that we want to treat these elements as an implicit list 
    // so they can be converted to List of objects. 
    @XStreamImplicit(itemFieldName = "ban") 
    private List bans = new ArrayList();
}

@XStreamAlias("BAN") // another mapping
public class Ban {
    
    /*
     We want to have different field names in Java classes so
     we define what element should be mapped to each field
    */
    @XStreamAlias("UPDATED_AT") // 
    private String dateOfUpdate;

    @XStreamAlias("TROUBLEMAKER")
    private Person person;
}

@XStreamAlias("TROUBLEMAKER")
public class Person {

    @XStreamAlias("NAME1")
    private String firstName;

    @XStreamAlias("NAME2")
    private String lastName;

    @XStreamAlias("AGE") // String will be auto converted to int value
    private int age;

    @XStreamAlias("NUMBER")
    private String documentNumber;
[/java]

And actual parsing logic is very short:

[java]
    FileReader reader = new FileReader("file.xml");  // load file

    XStream xstream = new XStream();
    xstream.processAnnotations(Data.class);     // inform XStream to parse annotations in Data class
    xstream.processAnnotations(Ban.class);      // and in two other classes... 
    xstream.processAnnotations(Person.class);   // we use for mappings
    Data data = (Data) xstream.fromXML(reader); // parse
 
    // Print some data to console to see if results are correct
    System.out.println("Number of bans = " + data.getBans().size());
    Ban firstBan = data.getBans().get(0);
    System.out.println("First ban = " + firstBan.toString()); 
[/java]

As you can see annotations are very easy to use and as a result final code is very concise. But what to do in situation when we can't modify mapping classes? We can use different approach that doesn't require any modifications in Java classes representing XML data.



#### Parse without annotations


When we can't enrich our model classes with annotations, there is another solution. We can define all mapping details using methods from XStream object:

[java]
    FileReader reader = new FileReader("file.xml");  // three first lines are easy, 
    XStream xstream = new XStream();                 // same initialisation as in the 
    xstream.alias("DATA", Data.class);               // basic example above

    xstream.alias("BAN", Ban.class);             // two more aliases to map... 
    xstream.alias("TROUBLEMAKER", Person.class); // between node names and classes

    // We want to have different field names in Java classes so
    // we have to use aliasField(<fieldInXml>, <mappedJavaClass>, <mappedFieldInJavaClass>)
    xstream.aliasField("UPDATED_AT", Ban.class, "dateOfUpdate"); 
    xstream.aliasField("TROUBLEMAKER", Ban.class, "person");

    xstream.aliasField("NAME1", Person.class, "firstName");
    xstream.aliasField("NAME2", Person.class, "lastName");
    xstream.aliasField("AGE", Person.class, "age");  // notice here that XML will be auto-converted to int "age"
    xstream.aliasField("NUMBER", Person.class, "documentNumber");
    
    /*
     Another way to define implicit collection
    */ 
    xstream.addImplicitCollection(Bans.class, "bans");

    Data data = (Data) xstream.fromXML(reader);  // do the actual parsing

    // let's print results to check if data was parsed
    System.out.println("Number of bans = " + data.getBans().size());
    Ban firstBan = data.getBans().get(0);
    System.out.println("First ban = " + firstBan.toString());
[/java]

As you can see XStream allows to easily convert more complicated XML structures into Java objects, it also gives a possibility to tune results by using different names if this from XML doesn't suit our needs.
But there is one thing should catch your attention: we are converting XML representing a Date into raw String which isn't quite what we would like to get as a result. That's why we will add converter to do some job for us.



### Using existing custom type converter


XStream library comes with set of built converters for most common use cases. We will use DateConverter. So now our class for Ban looks like that:
[java]
public class Ban {

    private Date dateOfUpdate;
    private Person person;
}
[/java]
And to use DateConverter we simply have to register it with date format that we expect to appear in XML data:
[java]
   xstream.registerConverter(new DateConverter("yyyy-MM-dd", new String[] {}));
[/java]

and that's it. Now instead of String our object is populated with Date instance. Cool and easy! But what about classes and situations that aren't covered by existing converters? We could write our own.



### Writing custom converter from scratch


Assume that instead of dateOfUpdate we want to know how many days ago update was done:
[java]
public class Ban {
    private int daysAgo;
    private Person person;
}
[/java]
Of course we could calculate it manually for each Ban object but using converter that will do this job for us looks more interesting. Our DaysAgoConverter must implement [Converter](http://xstream.codehaus.org/javadoc/com/thoughtworks/xstream/converters/Converter.html) interface so we have to implement three methods with signatures looking a little bit scary:
[java]
public class DaysAgoConverter implements Converter {
    @Override
    public void marshal(Object source, HierarchicalStreamWriter writer, MarshallingContext context) {
    }

    @Override
    public Object unmarshal(HierarchicalStreamReader reader, UnmarshallingContext context) {
    }

    @Override
    public boolean canConvert(Class type) {
        return false;
    }
}
[/java]
Last one is easy as we will convert only Integer class. But there are still two methods left with these HierarchicalStreamWriter, MarshallingContext, HierarchicalStreamReader and UnmarshallingContext parameters. Luckily, we could avoid dealing with them by using [AbstractSingleValueConverter](http://xstream.codehaus.org/javadoc/com/thoughtworks/xstream/converters/basic/AbstractSingleValueConverter.html)that shields us from so low level mechanisms. And now our class looks much better:
[java]
public class DaysAgoConverter extends AbstractSingleValueConverter {

    @Override
    public boolean canConvert(Class type) {
        return type.equals(Integer.class);
    }

    @Override
    public Object fromString(String str) {
        return null;
    }

    public String toString(Object obj) {
        return null;
    }
}
[/java]
Additionally we must override method toString(Object obj) defined in AbstractSingleValueConverter as we want to store Date in XML calculated from Integer, not a simple _Object.toString_ value which would be returned from default _toString_ defined in abstract parent. 

**Implementation**

Code below is pretty straightforward, but most interesting lines are commented. I've skipped all validation stuff to make this example shorter.
[java]
public class DaysAgoConverter extends AbstractSingleValueConverter {

    private final static String FORMAT = "yyyy-MM-dd"; // default Date format that will be used in conversion
    private final DateTime now = DateTime.now().toDateMidnight().toDateTime(); // current day at midnight

    public boolean canConvert(Class type) {
        return type.equals(Integer.class);     // Converter works only with Integers
    }

    @Override
    public Object fromString(String str) {
        SimpleDateFormat format = new SimpleDateFormat(FORMAT);
        try {
            Date date = format.parse(str);
            return Days.daysBetween(new DateTime(date), now).getDays();  // we simply calculate days between using JodaTime
        } catch (ParseException e) {
            throw new RuntimeException("Invalid date format in " + str);
        }
    }

    public String toString(Object obj) {
        if (obj == null) {
            return null;
        }

        Integer daysAgo = ((Integer) obj);
        return now.minusDays(daysAgo).toString(FORMAT); // here we subtract days from now and return formatted date string
    }
}
[/java] 

**Usage**
To use our custom converter for a specific field we have to inform about it XStream object using registerLocalConverter:
[java]
    xstream.registerLocalConverter(Ban.class, "daysAgo", new DaysAgoConverter());
[/java]
We are using "local" method to apply this conversion only to specific field and not to every Integer field in XML file. And after that we will get our Ban objects populated with number of days instead of Date.



### Summary


That's all what I wanted to show you in this post. Now you have basic knowledge about what XStream is capable of and how it can be used to easily map XML data to Java objects If you need something more advanced, please check [project official page](http://xstream.codehaus.org/) as it contains very good documentation and examples. 
