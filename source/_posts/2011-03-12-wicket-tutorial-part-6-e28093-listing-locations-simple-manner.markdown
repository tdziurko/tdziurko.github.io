---
author: Tomasz Dziurko
comments: true
date: 2011-03-12 18:58:47+00:00
layout: post
slug: wicket-tutorial-part-6-%e2%80%93-listing-locations-simple-manner
title: Wicket Tutorial, part 6 – listing locations in a simple manner
wordpress_id: 615
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

Today we will add simple listing view for locations added to the database in previous posts.  I said 'simple' as there are some more sophisticated ways to show many items using Wicket but probably we will not have many locations to show in our application so simple view without pagination should be ok. More complicated way ([DataTable](http://wicketstuff.org/wicket14/repeater/?wicket:bookmarkablePage=:org.apache.wicket.examples.repeater.DataTablePage) component) will be covered in the future when we will add 'listing items' feature'.

<!-- more -->But first small bug I found in our DAO layer. In class AbstractDAO we created method findByProperty and there is one line there:

[java]

return getEntityManager().createQuery(findEntityQuery).getSingleResult();

[/java]

At first everything  seems to be ok, but what will happen when query returns no result? An ugly exception is thrown. So we have to fix it with following code:

[java]
TypedQuery<EntityClass> query = getEntityManager().createQuery(findEntityQuery);
		List<EntityClass> entities = query.setMaxResults(1).getResultList();

		if(entities.isEmpty()) {
			return null;
		}
		else {
			return entities.get(0);
		}
[/java]

After this one, we could return to main topic of this post.


### Preparing logic


As we need list of all locations we must add one method to ListingService interface and class:

[java]
public class LocationServiceImpl implements LocationService {

	@Autowired
	private LocationDao locationDao;

	// (...)

	public List<Location> findAll() {
		return locationDao.findAll();
	}

}
[/java]

LocationDao uses generic method findAll from AbstractDAO to load all objects from db.


### Adding listing component


When we are ready with our logic, we could look at LocationsPage.java and HTML files:

First we add placeholders for Wicket objects in HTML:
- _locations_ for component to list all locations from database. We add it around tr element so each item will be rendered as separate table row. Of course you can use li, span, div instead.
- _id_: a label **inside** locations to render ID of each entity object
- _name_: a label **inside** locations to show name of each location

[html]
<body>
	<wicket:extend>
		<h3>Locations</h3>
		<br/>
			<table>
				<tr><th>ID</th><th>Name</th></tr>
				<span wicket:id="locations">
					<tr>
						<td><span wicket:id="id"></span></td>
						<td><span wicket:id="name"></span></td>
					</tr>
				</span>
			</table>
		<div style="float: right;">
			<a href="#" wicket:id="addLocationPageLink">Add new location</a>
		</div>
	</wicket:extend>
</body>
[/html]

Then we modify wicket page class:

[java]
public class LocationsPage extends BasePage {

	//   (...)

	private void initGui() {
		addLocationsModule();                               // (1)
		addCreateNewLocationLink();
	}

	private void addLocationsModule() {
		ListView<Location> locations = new ListView<Location>("locations", createModelForLocations()) { // (2)
			@Override
			protected void populateItem(ListItem<Location> item) {             // (3)
				item.add(new Label("id", new PropertyModel<Location>(item.getModel(), "id")));    // (4)
				item.add(new Label("name", new PropertyModel<Location>(item.getModel(), "name")));
			}
		};

		add(locations);
	}

	private LoadableDetachableModel<List<Location>> createModelForLocations() {    // (5)

		return new LoadableDetachableModel<List<Location>>() {    // (6)

			@Override
			protected List<Location> load() {    // (7)
				return locationService.findAll();
			}
		};
	}

}
[/java]

So what is happening in the code above? Let me explain it in more detail. In **(1)** we add new method to initGui(), in** (2)** we create [ListView](http://wicket.apache.org/apidocs/1.4/org/apache/wicket/markup/html/list/ListView.html) component which renders list of object in a selected way. In **(2)** you can also notice a createModelForLocations() method invocation, but this method will be covered a little later. ListView class has one method we have to implement: populateItem **(3)** which is called for each object from collections or collection model passed to ListView.

In this method we must add two labels declared inside locations element in HTML file, and we do it in **(4)**.  And here, as authors of Wicket and developers using this framework suggest, we use model instead of sending pure String value to the component. Why? Because if we use property of entity loaded from database, we are connecting view with this data and when page state is serialized also entity object is serialized too. And if our application uses Open Session in View we could end with the whole object graph serialized which will dramatically increase memory needed for each user session. So this is not what we really want :)

In **(4)** we are using [PropertyModel](http://wicket.apache.org/apidocs/1.4/org/apache/wicket/model/PropertyModel.html) which wraps Location model to get one property from it, id or name. Model wrapping is one of the best ways to use Wicket models effectively. Thanks to it we have only one Location model and we can wrap it many times in many ways to use it in a different situations.

Now, it's time to talk about mysterious method creating model for our ListView component **(5)**:

[java]
	private LoadableDetachableModel<List<Location>> createModelForLocations() {

		return new LoadableDetachableModel<List<Location>>() {

			@Override
			protected List<Location> load() {
				return locationService.findAll();
			}
		};
	}
[/java]

In this method we are using [LoadableDetachableModel](http://wicket.apache.org/apidocs/1.4/org/apache/wicket/model/LoadableDetachableModel.html), another feature which you should use often if you want your applications to be memory efficient. This model (more detailed info can be found [here](https://cwiki.apache.org/WICKET/detachable-models.html)) has one nice feature, it nullifies model object at the end of each request and loads it when needed. So no risk to have your 100000 entity objects serialized when page is saved on the server.


### Results




->![](/images/blog/2011/03/SS-20110312191713.png)<-




Working as expected. But if we view this page with empty database it will look rather strange:







->![](/images/blog/2011/03/SS-20110312192333.png)<-




ListView component isn't rendered (no locations so nothing to show) but as it is placed inside table and under row with headers this part of HTML is visible.  We must fix it and additionally, we will add with information label when there are no locations.




The simplest way of hiding component is to call setVisible(false) method but as we want to hide pure HTML which isn't directly assigned to any Wicket component, this way won't work. In such cases framework provides _wicket:enclosure_ tag (read more about Wicket tags [here](https://cwiki.apache.org/WICKET/wickets-xhtml-tags.html)) which renders html inside only when connected Wicket component is visible. Let me show this in practice:



[html]
		<wicket:enclosure child="locations">
			<table>
				<tr><th>ID</th><th>Name</th></tr>
				<span wicket:id="locations">
					<tr>
						<td><span wicket:id="id"></span></td>
						<td><span wicket:id="name"></span></td>
					</tr>
				</span>
			</table>
		</wicket:enclosure>
[/html]

We simply surround table with wicket:enclosure and define its child component as locations. So when locations.getVisible() returns false, no code inside enclosure is rendered.
Now we need to make location ListView component invisible when list is empty:

[java]
        locations.setVisible(!locations.getList().isEmpty());
[/java]

Moreover we want to show information when there are no locations, so we add label:

[html]
<body>
	<wicket:extend>
		<h3>Locations</h3>
		<br/>
		<wicket:enclosure child="locations">
			<table>
				<tr><th>ID</th><th>Name</th></tr>
				<span wicket:id="locations">
					<tr>
						<td><span wicket:id="id"></span></td>
						<td><span wicket:id="name"></span></td>
					</tr>
				</span>
			</table>
		</wicket:enclosure>

		<div wicket:id="noLocationsLabel"></div> <!--  NEW label here -->
		<div style="float: right;">
			<a href="#" wicket:id="addLocationPageLink">Add new location</a>
		</div>
	</wicket:extend>
</body>
[/html]

and in Java page class we create Label which is visible when ListView is not:

[java]
		Label noLocationsLabel = new Label("noLocationsLabel", "There are no locations in the database. Maybe you can add one?");
		noLocationsLabel.setVisible(!locations.isVisible());
		add(noLocationsLabel);
[/java]

After these improvements our page is ready:

->![](/images/blog/2011/03/SS-20110312194653.png)<-

Page is ready, you are ready to use ListView component in your projects  so we reached the end of this post. As usual latest source code is available [here](https://bitbucket.org/tdziurko/item-directory/src) and changeset from this post (except DAO bug fix which was pushed earlier) can be viewed [here](https://bitbucket.org/tdziurko/item-directory/changeset/2403961c4a2b).
