---
author: Tomasz Dziurko
comments: true
date: 2011-02-17 21:51:42+00:00
layout: post
slug: wicket-tutorial-part-4-custom-validator-unique-entity
title: Wicket Tutorial, part 4 - custom validator for unique entity name
wordpress_id: 524
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

Welcome back to our [Wicket tutorial](http://tomaszdziurko.pl/category/wicket-tutorial/) series! :) As I mentioned in [previous post](http://tomaszdziurko.pl/2011/02/wicket-tutorial-part-3-form-application/), today we will concentrate on building validator stopping user from adding item location with non-unique name. At first we will start with non-generic but working validator and then we will try to make is as flexible as possible. Ok, so let's write some code!


### Problem


As you might remember, currently we have Wicket form with basic validation checking existence of name field and whether its length is in proper range or not:

[java]
		Form<AddLocationPage> addLocationForm = new Form<AddLocationPage>("addLocationForm",
				new CompoundPropertyModel<AddLocationPage>(this));
		add(addLocationForm);

		Label nameLabel = new Label("nameLabel", "Location name");
		addLocationForm.add(nameLabel);

		RequiredTextField<String> nameField = new RequiredTextField<String>("name");
		nameField.add(LengthBetweenValidator.lengthBetween(MIN_LOCATION_NAME_LENGTH, MAX_LOCATION_NAME_LENGTH));
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
[/java]


<!-- more -->


But with unique constraint on name field in locations table in our database, trying to insert new location with name equal to other existing location will, as expected, end with such error:


[![](/images/blog/2011/02/SS-20110215205613.png)](/images/blog/2011/02/SS-20110215205613.png)





### Solution


The easiest way to prevent user from causing such exceptions is to create custom validator checking if such location is already in our database.  And as we will be validating String input we will use [StringValidator](http://wicket.apache.org/apidocs/1.4/index.html?org/apache/wicket/validation/validator/StringValidator.html) class as base class for our own validator called UniqueLocationNameValidator:

[java]
public class UniqueLocationNameValidator extends StringValidator {

	@SpringBean
	private LocationService locationService;    // (1)

	private long locationIdToIgnore;            // (2)

	public UniqueLocationNameValidator(long locationIdToIgnore) {
		this();
		this.locationIdToIgnore = locationIdToIgnore;
	}

	public UniqueLocationNameValidator() {
		InjectorHolder.getInjector().inject(this);            // (3)
	}

	@Override
	protected void onValidate(IValidatable<String> validatable) {      // (4)
		Location location = locationService.findByName(validatable.getValue());

		if(location != null && location.getId().longValue() != locationIdToIgnore) {
			error(validatable);
		}
	}
}
[/java]

So starting from the beggining of listing above:



	
  1. As we will need service object to communicate with locations database table, we are injecting LocationService

	
  2. In the future, we will also use this validator to validate locations while editing them. Id of currently edited object will prevent validator from firing error when found object will have the same Id as edited one. Currently we will not use this feature.

	
  3. @SpringBean annotation is working without any other effort only in Wicket [Component](http://wicket.apache.org/apidocs/1.4/index.html?org/apache/wicket/Component.html) class and its children so when we want to use it in different class, we have to tell Wicket to auto inject beans here.

	
  4. onValidate() method is the core of validating class. We try to search for location with the same name as provided by user and if application finds one and its Id is different from edited entity object, we fire an error message.


Error message can be defined in properties file and, as we will probably use our validator in more than one place, we add it in Application.properties file:

[xml]
applicationName = Item Directory
applicationHeader = Manage your items easily!
UniqueLocationNameValidator = Location with this name already exists in the database.
[/xml]

Of course we have to create findByName() method in LocationService. In our layered architecture (which currently could be considered as overkill but in larger project it really pays off) service calls dao method, so logic for finding location by name is in LocationDaoImpl class:

[java]
public Location findByName(String locationName) {
		Query findByNameQuery = entityManager.createQuery("from " + getClazz().getSimpleName()
                    + " entity where entity.name = :aName");
		findByNameQuery.setParameter("aName", locationName);

		return (Location) findByNameQuery.getSingleResult();

	}
[/java]

When we have our validator and DAO logic ready, we can modify Wicket form to make it complete and ready for the production. To use custom validator we simply create such object and add it to field we want to validate (fragment of AddLocationPage class below):

[java]
		UniqueLocationNameValidator locationNameValidator = new UniqueLocationNameValidator();
		nameField.add(locationNameValidator);
[/java]


And when we submit this form with existing location name we will see feedback message showing that validation is working:




[![](/images/blog/2011/02/SS-20110215220102.png)](/images/blog/2011/02/SS-20110215220102.png)Ok, this is a good moment for a short break. If you want to see complete changes we made so far in the code, please refer to the this [changeset](https://bitbucket.org/tdziurko/item-directory/changeset/15bf0c549e64) in BitBucket repository. And now at this point we could end this post, as we reached our goal for today: validation is complete and running. But there is one problem...





### We are lazy :)




Yeah, we are lazy, really lazy developers and we hate breaking [DRY](http://pl.wikipedia.org/wiki/DRY) rule, so instead of writing N unique entity validators for each time we need one, we will write generic one. But more about this in the next  (almost ready) post :)




Thank you for reading and see you next time. Please don't forget to leave your feedback in the comments.









