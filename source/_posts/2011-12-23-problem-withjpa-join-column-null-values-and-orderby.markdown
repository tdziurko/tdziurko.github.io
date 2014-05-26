---
author: Tomasz Dziurko
comments: true
date: 2011-12-23 17:27:56+00:00
layout: post
slug: problem-withjpa-join-column-null-values-and-orderby
title: 'Problem with JPA, join column with null values and order by '
wordpress_id: 1176
categories:
- How-to
- Java
tags:
- JPA2
- JPQL
- null property
- order by
- sorting
---

Recently I've encountered a tricky and hard to spot problem with JPA2, joins with null values and ordering. But finally I managed to solve it and as I couldn't find anything related to this issue in Google, I think it is worth to share our working solution.

Problem occurs when we try to sort list of users using something like "_from User u order by u.country.name desc_" and sometimes country is null. In such case entities without country are simply disappearing from our result list. Longer description of this issue and how we solved it can be read below.

<!-- more -->


### Problem


Let's say we have three entities: User, City and Country. User has a City and also has a Country. City and Country are not connected in any way to make this example less complicated.

``` java
@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "login", length = 255, nullable = false)
    private String login;

    @Column(name = "name", length = 255, nullable = false)
    private String name;

    @Column(name = "surname", length = 255, nullable = false)
    private String surname;

    @ManyToOne(optional = true, cascade = CascadeType.ALL)
    private Country country;

    @ManyToOne(optional = true, cascade = CascadeType.ALL)
    private City city;
}
```



``` java
@Entity
@Table(name = "city")
public class City {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "name", length = 255, nullable = false)
    private String name;
}
```



``` java
@Entity
@Table(name = "country")
public class Country {

    @Id
    @GeneratedValue
    private Long id;

    @Column(name = "name", length = 255, nullable = false)
    private String name;

    @Column(name = "population", nullable = false)
    private Long population;
}
```

What we want to achieve is to load all users and display their details in the datatable. We want to show user name, surname and also some data from city (its name) and  country (name as well and population). User can fill this data in his profile page but it is not mandatory so we might have user without city and/or country in our database. Of course this datatable on the web page should be sortable using all displayed properties. For  such functionality we need a service class:

``` java
@Transactional
public class UserService {

    @Writeable
    @Inject
    private EntityManager entityManager;

    public List<User> loadAll(UserSortField sortField, boolean ascending) {

        String queryString = "from User u order by u." + sortField.getField() + (ascending ? " asc " : " desc ");
        return entityManager.createQuery(queryString).getResultList();
    }
}
```

and enum defining available _order by_ options

``` java
public enum UserSortField {

    ID("id"),
    NAME("name"),
    SURNAME("surname"),
    COUNTRY_POPULATION("country.population"),
    COUNTRY_NAME("country.name"),
    CITY_NAME("city.name");

    private String field;

    private UserSortField(String field) {
        this.field = field;
    }

    public String getField() {
        return field;
    }
}
```

So far everything looks just fine. So let's write some tests. As we are using JEE6, tests will be written using Arquillian and utility class AbstractDBTest from [softwaremill-common](https://github.com/softwaremill/softwaremill-common). More about this class and how to use it can be found in [Adam Warski's blog](http://www.warski.org/blog/2010/11/db-test-run-tests-with-a-database-using-arquillian/). And believe me, it makes testing with Arquillian so, so much easier.

Our test class looks like that:

``` java
public class UserServiceTest extends AbstractDBTest {

    @Inject
    private UserService userService;

    @Deployment
    public static JavaArchive createTestArchive() {
        return new ArchiveConfigurator() {
            @Override
            protected JavaArchive configureBeans(JavaArchive ar) {
                return ar.addPackage(UserService.class.getPackage());
            }
        }.createTestArchive();
    }

    @Override
    protected void configureEntities(Ejb3Configuration cfg) {
        cfg.addAnnotatedClass(User.class);
        cfg.addAnnotatedClass(City.class);
        cfg.addAnnotatedClass(Country.class);
    }

    @Override
    protected void loadTestData(EntityManager entityManager) {

        // here we prepare test data with 10 users.
        // Some of them have city and country defined, some of them don't
    }

    @Test
    public void shouldLoadAllUsers() throws Exception {
        assertThat(userService.loadAll(UserSortField.ID, true).size()).isEqualTo(10);
    }

    @Test
    public void shouldOrderByUserName() throws Exception {
        List<User> allUsers = userService.loadAll(UserSortField.NAME, true);
        assertThat(allUsers.size()).isEqualTo(10);
        assertThat(allUsers).onProperty("name").containsSequence(
                "name_01", "name_02", "name_03", "name_04", "name_05", "name_06",
                "name_07", "name_08", "name_09", "name_10");
    }
}
```

These both tests will pass. But let's add another one with order by country name:

``` java
    @Test
    public void shouldOrderByCountryName() throws Exception {
        List<User> allUsers = userService.loadAll(UserSortField.COUNTRY_NAME, true);
        assertThat(allUsers.size()).isEqualTo(10);
        assertThat(allUsers).onProperty("country.name").containsSequence(
                "Belgium", "Poland", "Poland", "Poland", "Poland", "South Africa",
                "Spain", "Togo", null, null);
    }
```

And when we re-run, we will see that this one is failing with error saying that expected number elements was 10 but actual is 8.

Ok, so we have a problem now. We have a query that should work just fine but apparently JPA omits entities with null property when we try to sort list using it.


### Solution


To solve this problem we have to approach it from a different side. Instead of returning a list of users with countries and cities we could return a descriptor containing all the data we need to display in the database. So let's create this class:

``` java
public class UserDescriptor {

    private String name;
    private String surname;
    private String countryName;
    private Long countryPopulation;
    private String cityName;

    public UserDescriptor(String name, String surname, String countryName, Long countryPopulation, String cityName) {
        this.name = name;
        this.surname = surname;
        this.countryName = countryName;
        this.countryPopulation = countryPopulation;
        this.cityName = cityName;
    }

    // getters
}
```

and then adjust method in UserService:

``` java
public List<UserDescriptor> loadAllUserDescriptors(UserSortField sortField, boolean ascending) {

    String queryString = "select new pl.tomaszdziurko.UserDescriptor(u.name, u.surname, " +
            " country.name, country.population, " +
            " city.name) from User u left join u.city as city left join u.country as country " +
            " order by " + sortField.getField() + (ascending ? " asc " : " desc ");
    return entityManager.createQuery(queryString).getResultList();
}
```

This code fragment needs longer explanation. So basically for each returned row we create new object, the UserDescriptor. It consists of all data we need to display in our datatable. But instead of writing "_u.country.name, u.country.population_" we type "_country.name, country.population_" and add proper joins to the query. Of course our SortFieldEnum needs little attention too as we must tune sort fields a bit to be in line with modified query.

After these steps we are ready to make our tests green. But when we re-run them we will see following error message:

    
    java.lang.AssertionError: list:<[null, null, 'Belgium', 'Poland', 'Poland', 'Poland', 'Poland', 'South Africa', 'Spain', 'Togo']> 
    does not contain the sequence: <['Belgium', 'Poland', 'Poland', 'Poland', 'Poland', 'South Africa', 'Spain', 'Togo', null, null]>


So now we have 10 items in our list so there are no ommited ones but another problem appeared. Null values are at the beggining. If it is not a problem for you, this could be the end of this post. If you want the know how to solve this, keep reading.

**Moving nulls to proper location**

If we want to have nulls at the end of our result list, we must add new rule in_ order by _clause. So our query needs some tweaking:

``` java
public List<UserDescriptor> loadAllUserDescriptors(UserSortField sortField, boolean ascending) {

    String queryString = "select new pl.tomaszdziurko.UserDescriptor(u.name, u.surname, " +
            " country.name, country.population,  city.name, " +
            " case when " + sortField.getEntity() + " is null then 2 else 1 end as nullOrderer) from User u " +
            " left join u.city as city left join u.country as country " +
            " order by nullOrderer " + (ascending ? " asc " : " desc ") + ", " +
            sortField.getField() + (ascending ? " asc " : " desc ");

    return entityManager.createQuery(queryString).getResultList();
}
```

The first thing worth to notice is that we've added _case when " + sortField.getEntity() + " is null then 2 else 1 end_ _as nullOrderer_. And we use _nullOrderer_ as a first element in _order by_ clause. Thanks to this, our records are sorted first by not null and then using requested field. Moreover as you can see, our sort enum was enhanced with new property, an _entity_, which is used to define on which object we want to check nullability. For fields from User entity it will be User alias _u_, for Country _country _and for City _city._ So now enum looks like that:

``` java
public enum UserSortField {

    ID("u", "u.id"),
    NAME("u", "u.name"),
    SURNAME("u", "u.surname"),
    COUNTRY_POPULATION("country", "country.population"),
    COUNTRY_NAME("country", "country.name"),
    CITY_NAME("city", "city.name");

    private String entity;
    private String field;

    private UserSortField(String entity, String field) {
        this.entity = entity;
        this.field = field;
    }

    public String getEntity() {
        return entity;
    }

    public String getField() {
        return field;
    }
}
```

And when we run our failing test, everything looks just perfect and green. But to make sure that we didn't omit anything, let's add a few more testing methods:

``` java
@Test
public void shouldOrderByCityName() throws Exception {
    List<UserDescriptor> allUsers = userService.loadAllUserDescriptors(UserSortField.CITY_NAME, true);
    assertThat(allUsers).onProperty("cityName").containsExactly(
            "Antwerp", "Cracow", "Madrid", "Pretoria", "Warsaw", "Warsaw",
             null, null, null, null);
}

@Test
public void shouldOrderByCountryPopulation() throws Exception {
    List<UserDescriptor> allUsers = userService.loadAllUserDescriptors(UserSortField.COUNTRY_POPULATION, false);
    assertThat(allUsers).onProperty("countryPopulation").containsExactly(
            null, null, 51L, 46L, 38L, 38L, 38L, 38L, 11L, 6L);
}
```

**Summary**

So after some struggles we have working query returning proper results and sorting objects as client requested.  And what is more important, we were able to test it with Arquillian in an embedded Weld container.

All sources are available in my [GitHub repository](https://github.com/tdziurko/nullable-property)
