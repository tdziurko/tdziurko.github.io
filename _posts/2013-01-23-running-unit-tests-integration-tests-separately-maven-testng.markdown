---
layout: post
title: Running unit tests and integration tests separately with Maven and TestNG
description: Configuring Maven Failsafe plugin to run TestNG integration tests separately from unit tests
date: 2013-01-23
author: tomek
image: '/images/maven-testng-integration-tests-cover.jpg'
tags: [Programming, Java]
tags_color: '#5fb05f'
redirect_from:
  - /2013/01/running-unit-tests-integration-tests-separately-maven-testng/
  - /2013/01/running-unit-tests-integration-tests-separately/
---

Recently for my new pet project I decided that I would like to have some tests executed during the standard `mvn test` and some other ones only during a different phase, let’s call it integration phase. I googled and googled and nothing seemed to work, so after struggling with making my setup work I’ve decided to write down my findings on how I was able to configure TestNG with Maven to run integration and unit tests separately.

## Basic (not working) setup

For integration testing there is a [Maven Failsafe plugin](http://maven.apache.org/surefire/maven-failsafe-plugin/integration-test-mojo.html) that is supposed to do what we want out of the box. Unfortunately, things are not as easy and straightforward as we might expect.

Let’s look at a basic setup that should work:

We have two test classes, one with IT postfix that will indicate that this is our integration test:

```java
import org.testng.annotations.Test;

import static org.fest.assertions.Assertions.assertThat;

public class ExampleUnitTest {

    @Test
    public void shouldPass() {
        assertThat(false).isFalse();
    }
}
```

and

```java
import org.testng.annotations.Test;

import static org.fest.assertions.Assertions.assertThat;

public class StatusTestIT {

    @Test
    public void shouldFail() {
        assertThat("aaa").isEqualTo("");
    }
}
```

and now let’s declare the plugin mentioned above in our pom.xml:

```xml
<project>

    <!-- ... -->
    <build>
        <plugins>
            <!-- ... -->
            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-surefire-plugin</artifactId>
                <version>2.9</version>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-failsafe-plugin</artifactId>
                <version>2.13</version>
                <configuration>
                    <includes>
                        <include>**/*IT.java</include>
                    </includes>
                </configuration>
            </plugin>
        </plugins>
    </build>
</project>
```

After that we should see two things. `mvn test` should pass as we have one green test in the ExampleUnitTest class, but `mvn failsafe:integration-test` should fail with StatusTestIT red. But as I said before, this does not work. `mvn test` looks as expected, but the second Maven execution is passing as well showing that no tests were run. Plugin seems to omit our completely valid test…

## Small fix doing its job

After testing different approaches I found out that all we need to make this setup work is to add the plugin execution to the integration-test phase of the maven life cycle. So tiny change, but now our integration tests are executed only when we call `mvn integration-test`.

```xml
<plugin>
    <groupId>org.apache.maven.plugins</groupId>
    <artifactId>maven-failsafe-plugin</artifactId>
    <version>2.13</version>
    <configuration>
        <includes>
            <include>**/*IT.java</include>
        </includes>
    </configuration>
    <executions>
        <execution>
            <id>failsafe-integration-tests</id>
            <phase>integration-test</phase>
            <goals>
                <goal>integration-test</goal>
            </goals>
        </execution>
    </executions>
</plugin>
```

And that’s all, we could execute integration tests only when we really want to. Of course there are some things to remember when adopting the setup described above:

- our unit tests must have Test postfix, if you use something else, we have to configure the surefire plugin using _include_
- during integration test unit tests are executed too. This is a small flaw in this setup, but exclude `**/*Test.java` seems not to work 🙁

Source code of this example setup is available on [GitHub](https://github.com/tdziurko/maven-testng-integration-tests)
