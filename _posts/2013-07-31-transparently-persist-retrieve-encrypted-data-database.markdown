---
layout: post
title: Transparently persist and retrieve encrypted data in the database
description: Automatic encryption and decryption of entity fields with Jasypt and Hibernate
date: 2013-07-31
author: tomek
image: '/images/placeholder-jasypt-encryption.png'
tags: [Programming, Java]
tags_color: '#5fb05f'
---

Big brother is watching… always so why not encrypt everything. And then, auto-decrypt and encrypt our data stored in the Database? Here comes Jasypt.

It’s over two months since my last post here but June and July were extremely busy and intensive months this year. First, the organisation of [Confitura](http://confitura.pl/) (the biggest free conference for Java developers in Europe) was taking all of my free evenings and then, after quite a nervous period in hospital, our 2nd son was born. But now, I will try to blog regularly again, so please stay tuned.

In this post I will shortly describe how we can store encrypted data in our database and retrieve it as already decrypted in a simple, transparent way using the [Jasypt](http://www.jasypt.org/) library. Our use case will be to store Twitter Api credentials so they are safe in our database but still easy to retrieve and use to post updates in our timeline.

So what we have is a simple entity representing our settings item:

```java
@Entity
public class SettingsItem implements Serializable {

    @Id
    @GeneratedValue(strategy = javax.persistence.GenerationType.AUTO)
    private Integer id;

    private String name;
    private String encryptedValue;
}
```

In such table we will store values for Twitter Consumer Key, Twitter Access Token and so on.

What we would like to achieve is that when we create a SettingItem object with the value as plain text and then we persist it, encryption is automatically performed so in the DB we have an encrypted String. And of course, when we retrieve data from DB, we want to see a decrypted String without additional effort, just out of the box.

## Jasypt to the rescue

![jasypt](/images/jasypt.png)

[Jasypt](http://www.jasypt.org/) is a simple encryption library written in Java. It frees the developer from dealing with low level configuration details and makes the whole encryption process easy and straightforward. And what is most interesting now, it also has a nice integration with Hibernate allowing seamless encryption/decryption of data stored in the database.

### Setup

To use Jasypt and its Hibernate integration module we have to add two items to our pom:

```xml
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
```

### Custom type

Then we have to declare a custom Hibernate type (@TypeDef) in our entity:

```java
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
```

and after that in the same class we can mark our encryptedValue field to use this custom type:

```java
@Type(type="encryptedString")
private String encryptedValue;
```

### Registering encryptor

We are almost done. The last thing we have to do is register the encryptor in HibernatePBEEncryptorRegistry class. This can be done in the initialization class of our application, e.g. ServletContext or simply in the class with the main(String[] args) method:

```java
String password = System.getProperty("jasypt.password");

StandardPBEStringEncryptor strongEncryptor = new StandardPBEStringEncryptor();
strongEncryptor.setPassword(password);
HibernatePBEEncryptorRegistry registry =
        HibernatePBEEncryptorRegistry.getInstance();
registry.registerPBEStringEncryptor("STRING_ENCRYPTOR", strongEncryptor);
```

One important thing here is that by using System.getProperty() or System.getenv() we can safely configure our encryption mechanism, the password is provided at runtime by setting a proper value on the server machine.

## Summary

As a summary, one short passing test showing that our solution works:

```java
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
```
