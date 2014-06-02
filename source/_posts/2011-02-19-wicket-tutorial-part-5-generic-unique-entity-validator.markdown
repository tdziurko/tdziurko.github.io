---
author: Tomasz Dziurko
comments: true
date: 2011-02-19 23:04:04+00:00
layout: post
slug: wicket-tutorial-part-5-generic-unique-entity-validator
title: Wicket tutorial, part 5 - generic unique entity validator
wordpress_id: 559
keywords: Wicket, tutorial, Wicket tutorial, how to, start, learn, generic custom validator
categories:
- How-to
- Item Directory application
- Java
- Wicket
- Wicket Tutorial
tags:
- How-to
- Java
- Tutorial
- Wicket
---

As I promised in the [previous post](/2011/02/wicket-tutorial-part-4-custom-validator-unique-entity/), today we will focus on transforming our unique name validator in Wicket to generic one. So now, let's simply list what we are going to achieve;



	
  * validator should validate any entity object in our project

	
  * validator should check unique value of any property from validated object

	
  * error message should be generic too but customization should be easy


<!-- more -->


### Generic DAO element


First thing we need is common query to load any entity with chosen property name and value. So we start with adding findByProperty method to DAO interface:

``` java
public interface DAO<EntityClass extends IEntity> {

	List<EntityClass> findAll();

	EntityClass findById(Long id);

	EntityClass findByProperty(String propertyName, Object propertyValue);

	// (...)
}
```

Next we implement this method in base DAO class named AbstractDAO:

``` java
public abstract class AbstractDAO<EntityClass extends AbstractEntity>
        implements DAO<EntityClass> {

    // (...)

    public EntityClass findByProperty(String propertyName, Object propertyValue) {

    	if(propertyName == null || propertyValue == null) {
    		throw new IllegalArgumentException("Property name and value cannot be null! Values: propertyName" + propertyName + ", propertyValue = " + propertyValue);
    	}

    	CriteriaBuilder cb = getEntityManager().getCriteriaBuilder();     // (1)
		CriteriaQuery<EntityClass> findEntityQuery = cb.createQuery(getClazz());     // (2)
		Root<EntityClass> entity = findEntityQuery.from(getClazz());               // (3)

		Predicate propertyPredicate = cb.equal(entity.get(propertyName), propertyValue);   //  (4)

		findEntityQuery.where(propertyPredicate);       // (5)

		return getEntityManager().createQuery(findEntityQuery).getSingleResult();       // (6)
    }

}
```

Little explanation to the code above: we are using JPA2 Criteria Api which is one of the coolest new feature in the second edition of Java Persistence Api. As it's not a main topic of this post, don't expect detailed information about JPA2 here. I will just try to provide short description of each step:



	
  1. CritieriaBuilder is a factory for almost every method and object in Critieria API, so we use one here.

	
  2. CritieriaQuery specifies what we want to get from the database. In this case, we need object of class returned by getClazz() object.  getClazz() is a method from DAO interface which will be implemented in concrete DAO classes such as LocationDAOImpl.

	
  3. Root<EntityClass> represents SQL 'from' clause.

	
  4. Predicates represent elements after SQL 'where'.  We need only one, similar to ''where entity.propertyName = :propertyValue' so we create it with CriteriaBuilder.

	
  5. Predicate to do its job, must be added to query.

	
  6. Finally, we can return found results (or null).




### Generic validator


First step to create flexible validator is to rename it to more proper form. So now our class UniqueLocationNameValidator is UniqueEntityValidator. And since we are going to use classes other than only String for a validated property, we now can't extend [StringValidator](http://wicket.apache.org/apidocs/1.4/org/apache/wicket/validation/validator/StringValidator.html). Instead of it, we will use  [AbstractValidator](http://wicket.apache.org/apidocs/1.4/org/apache/wicket/validation/validator/AbstractValidator.html). But as it's generic we must introduce generic parameter too and signature of new class will look like this:

``` java

public class UniqueEntityValidator<PropertyClass> extends AbstractValidator<PropertyClass>

```

Then, as we are going to use any provided DAO instance, now we don't inject LocationService to the validator but pass  DAO implementing class for validated entity with the constructor. Another thing needed when constructing validator object is property name which will be used in validation process and also by the DAO to try to load existing entity.

``` java

public class UniqueEntityValidator<PropertyClass> extends AbstractValidator<PropertyClass> {

    private DAO<? extends IEntity> dao;
    private String propertyName;
    private long entityIdToIgnore;

    public UniqueEntityValidator(DAO<? extends IEntity> dao, long entityIdToIgnore, String propertyName) {
        this(dao, propertyName);
        this.entityIdToIgnore = entityIdToIgnore;
    }

    public UniqueEntityValidator(DAO<? extends IEntity> dao, String propertyName) {
        this.dao = dao;
        this.propertyName = propertyName;
    }

    // (...)
}
```

Ok, we are almost there. Let's see how the most important method in our validator look like:

``` java

@Override
protected void onValidate(IValidatable<PropertyClass> validatable) {
    IEntity entity = dao.findByProperty(propertyName, validatable.getValue());

    if(entity != null && entity.getId().longValue() != entityIdToIgnore) {
        error(validatable);
    }

}

```

When I started writing this generic validator, I thought that onValidate will be more complicated. But it appeared that it's very clean and simple solution (like most things in Wicket :) ). We just use findByProperty method from passed with constructor DAO and that's all.


### Generic error message


Every Wicket validator provides two elements that can be shown with error message:  label of validated component and user input. So in our example we create UniqueEntityValidator.properties file and place it next to validator class with one message inside:

``` xml
UniqueEntityValidator = Item with ${label} equal to '${input}' already exists.

```

Here we can see how label and input elements are used. Wicket in a runtime replaces these placeholders with proper data. Label as a default is a Wicket componentId value, but it can be customized with component.setLabel(new Model<String>("customLabel");

Of course this message can be changed in a simple way, we must only override method resourceKey() to return not validator class name but our new validator message key which will be used to find corresponding error message:

``` java
		UniqueEntityValidator<String> locationNameValidator = new UniqueEntityValidator<String>(locationDao, "name") {
			@Override
			protected String resourceKey() {
				return "CustomValidator";
			}
		};
                nameField.add(locationNameValidator);
```

New message (key CustomValidator with our custom error message value) should be placed in proper properties file: PageClass.properties or Application.properties in most cases.

And, because we reached end of this post, we could see how new validator is uded in AddLocationPage:

``` java
public class AddLocationPage extends BasePage {

	private static final int MIN_LOCATION_NAME_LENGTH = 5;
	private static final int MAX_LOCATION_NAME_LENGTH = 255;

	@SpringBean
	private LocationService locationService;

	@SpringBean
	private LocationDao locationDao;

	private String name;

	public AddLocationPage() {
		initGui();
	}

	private void initGui() {

		Form<AddLocationPage> addLocationForm = new Form<AddLocationPage>("addLocationForm",
				new CompoundPropertyModel<AddLocationPage>(this));
		add(addLocationForm);

		Label nameLabel = new Label("nameLabel", "Location name");
		addLocationForm.add(nameLabel);

		RequiredTextField<String> nameField = new RequiredTextField<String>("name");
		nameField.setLabel(new Model<String>("Name"));
		nameField.add(LengthBetweenValidator.lengthBetween(MIN_LOCATION_NAME_LENGTH, MAX_LOCATION_NAME_LENGTH));

        // here!
		UniqueEntityValidator<String> locationNameValidator = new UniqueEntityValidator<String>(locationDao, "name");
		nameField.add(locationNameValidator);

		addLocationForm.add(nameField);

		Button submitButton = new Button("submitButton") {
			@Override
			public void onSubmit() {
				Location location = new Location(name);
				locationService.save(location);

				getSession().info("Location added successfully");
				setResponsePage(LocationsPage.class);
			}
		};
		addLocationForm.add(submitButton);
	}
}
```

Generic String property allows validator to know that validatable.getValue() in UniqueEntityValidator should return String object and no casting is necessary. This rule applies to any validator we use in this way: unique Integer value, unique Date, etc.

We're done for today. Source code from this post can be viewed in [this changeset](https://bitbucket.org/tdziurko/item-directory/changeset/b92757c92ac2) in BitBucket repository and complete tutorial source is available [here](https://bitbucket.org/tdziurko/item-directory/src). At this moment I haven't made a decision what will be the topic of next post. Maybe you can suggest something what you find most interesting in Wicket and I will try to implement such feature in ItemDirectory application :)

Thank you for your time!