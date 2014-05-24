---
author: Tomasz Dziurko
comments: true
date: 2013-01-23 15:42:28+00:00
layout: post
slug: running-unit-tests-integration-tests-separately-maven-testng
title: Running unit tests and integration tests separately with Maven Failsafe and
  TestNG
wordpress_id: 1711
categories:
- How-to
- Java
tags:
- integration
- Maven
- testing
- TestNG
---

Recently for my new pet project I decided that I would like to have some tests executed during standard _mvn test_ and some other ones only during different phase, let's call it integration phase. I googled and googled and nothing seemed to work, so after struggling with making my setup work I've decided to write down my findings how I was able to configure TestNG with Maven to run integration and unit tests separately.



### Basic (not working) setup



For integration testing there is a [Maven Failsafe plugin](http://maven.apache.org/surefire/maven-failsafe-plugin/integration-test-mojo.html) that is supposed to do what we want out of the box. Unfortunately, things are not as easy and straightforward as we might expect.  
<!-- more -->

Let's look at basic setup that should work:

We have two test classes, one with IT postfix that will indicate that this is our integration test:

[java]
import org.testng.annotations.Test;

import static org.fest.assertions.Assertions.assertThat;

public class ExampleUnitTest {

    @Test
    public void shouldPass() {
        assertThat(false).isFalse();
    }
}
[/java]

and

[java]
import org.testng.annotations.Test;

import static org.fest.assertions.Assertions.assertThat;

public class StatusTestIT {

    @Test
    public void shouldFail() {
        assertThat("aaa").isEqualTo("");
    }
}
[/java]

and now let's declare plugin mentioned above in our pom.xml:

[xml]
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
[/xml]

After that we should see two things. _mvn test_ should pass as we have a one green test in ExampleUnitTest class, but _mvn failsafe:integration-test_ should fail with StatusTestIT red. But as I said before, this does not work. _mvn test_ looks as expected, but second Maven execution is passing as well showing that no test were run. Plugin seems to omit our completely valid test...



### Small fix doing its job



After testing different approaches I found out that all we need to make this setup work is to add plugin execution to an integration-test phase of maven life cycle. So tiny change, but now our integration tests are executed only when we call _mvn integration-test_.

[xml]
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
[/xml]

And that's all, we could execute integration tests only when we really want to. Of course there are some things to remember when adopting setup described above:



    
  * our unit tests must have Test postfix, if you use something else, we have to configure surefire plugin using _include_

    
  * during integration test unit tests are executed too. This is a small flaw in this setup, but exclude **/*Test.java seem not to work :( 



Source code of this example setup is available on [GitHub](https://github.com/tdziurko/maven-testng-integration-tests)

