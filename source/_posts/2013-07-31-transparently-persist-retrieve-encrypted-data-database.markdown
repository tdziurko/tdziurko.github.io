---
author: Tomasz Dziurko
comments: true
date: 2013-07-31 06:45:07+00:00
layout: post
slug: transparently-persist-retrieve-encrypted-data-database
title: Transparently persist and retrieve encrypted data from database
wordpress_id: 1863
categories:
- How-to
- Java
tags:
- encryption
- Hibernate
- Jasypt
- Java
---
<!-- more -->

It's over two months since my last post here but June and July were extremely busy and intensive months this year. First, organisation of [Confitura](http://confitura.pl) (the biggest free conference for Java developers in Europe) was taking all of my free evenings and then, after quite nervous period in hospital, our 2nd son was born. But now, I will try to blog regularly again, so please stay tuned.

In this post I will shortly describe how we can store encrypted data in our database and retrieve it as already decrypted in a simple, transparent way using [Jasypt](http://www.jasypt.org/) library. Our use case will be to store Twitter Api credentials so they are safe in our database but still easy to retrieve and use to post updates in our timeline.


So what we have is a simple entity representing our settings item:
[java]
@Entity
public class SettingsItem implements Serializable {

    @Id
    @GeneratedValue(strategy = javax.persistence.GenerationType.AUTO)
    private Integer id;
    
    private String name;
    private String encryptedValue;
}
[/java]

In such table we will store values for Twitter Consumer Key, Twitter Access Token and so on.

What we would like to achieve is that when we create SettingItem object with value as plain text and then we persist it, encryption is automatically performed so in the DB we have encrypted String. And of course, when we retrieve data from DB, we want to see decrypted String without additional effort, just out of the box.



### Jasypt to the rescue


->![jasypt-small](/images/blog/2013/07/jasypt-small.png)<-

[Jasypt](http://www.jasypt.org) is a simple encryption library written in Java. It frees developer from dealing with low level configuration details and makes whole encryption process easy and straightforward. And what is most interesting now, it also has nice integration with Hibernate allowing seamless encryption/decryption of data stored in database.



#### Setup


To use Jasypt and its Hibernate integration module we have to add two items to our pom:
[xml]
    <dependency>
        <groupId>org.jasypt</groupId>
        <artifactId>jasypt</artifactId>
        <version>1.9.0</version>
    </dependency>
    <dependency>
        <groupId>org.jasypt</groupId>
        <artifactId>jasypt-hibernate4</artifactId>
        <version>1.9.0</version>
    </dependency>
[/xml]



#### Custom type


Then we have to declare custom Hibernate type (@TypeDef) in our entity:

[java]
@TypeDef(
        name="encryptedString",
        typeClass=EncryptedStringType.class,
        parameters= {
                // value will be used later to register encryptor
                @Parameter(name="encryptorRegisteredName", value="STRING_ENCRYPTOR")
        }
)
@Entity
public class SettingsItem implements Serializable {
   // (...)
}
[/java]

and after that in the same class we can mark our encryptedValue field to use this custom type:

[java]
    @Type(type="encryptedString")
    private String encryptedValue;
[/java]



#### Registering encryptor


We are almost done. Last thing we have to do is register encryptor in HibernatePBEEncryptorRegistry class. This can be done in  initialization class of our application, e.g. ServletContext or simply in class with main(String[] args) method: 

[java]
    String password = System.getProperty("jasypt.password");

    StandardPBEStringEncryptor strongEncryptor = new StandardPBEStringEncryptor();
    strongEncryptor.setPassword(password);
    HibernatePBEEncryptorRegistry registry =
            HibernatePBEEncryptorRegistry.getInstance();
    registry.registerPBEStringEncryptor("STRING_ENCRYPTOR", strongEncryptor);
[/java] 

One important thing here is that by using System.getProperty() or System.getenv() we can safely configure our encryption mechanism, password is provided at runtime by setting proper value on server machine.



#### Summary


As as a summary, one short passing test showing that our solution works:

[java]
public class SettingsItemRepositoryShould extends IntegrationTest {

    @Autowired
    private SettingsItemRepository repository;


    @BeforeClass
    public static void init() {
        StandardPBEStringEncryptor strongEncryptor = new StandardPBEStringEncryptor();
        strongEncryptor.setPassword("JohnDoe");

        HibernatePBEEncryptorRegistry registry =
                HibernatePBEEncryptorRegistry.getInstance();
        registry.registerPBEStringEncryptor("STRING_ENCRYPTOR", strongEncryptor);
    }

    @Test
    public void shouldEncryptAndDecryptValue() {
        // Given
        String settingName = "test";
        String value = "EncryptMe";

        // When
        repository.save(new SettingsItem(settingName, value));

        // Then
        SettingsItem settingsItem = repository.findByName(settingName);
        assertThat(settingsItem.getEncryptedValue()).isEqualTo(value);
    }
}
[/java]
