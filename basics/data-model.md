# The LensKit Data Model

LensKit has a flexible data model that allows many different kinds of data to be loaded and operated on by algorithms.  This section describes that model.

## Entities

[Entity]: /apidocs/org/lenskit/data/entities/Entity.html
[CommonTypes]: /apidocs/org/lenskit/data/entities/CommonTypes.html
[CommonAttributes]: /apidocs/org/lenskit/data/entities/CommonAttributes.html
[TypedName]: /apidocs/org/lenskit/data/entities/TypedName.html

The heart of the LensKit data model is an *entity*, defined by the [Entity][] interface.  An entity is a particular piece of data, consisting of a *type* (the name of the type of data), an *id* (of type `long`), and zero or more named *attributes*.

Common entity types, whose type names are defined in [CommonTypes][], include:

- `user`, a user in the system
- `item`, an item in the system
- `rating`, a user-provided rating of an item

Attributes are identified by name, such as `user`; to improve reliability, `Entity` provides a type-safe interface, using [typed names][TypedName] that associate a type (such as `Long`) with a name.  [CommonAttributes][] provides constants defining typed names for many common attributes:

- `ENTITY_ID`, the entity's ID.
- `USER_ID`, a field (named `user`) storing a user ID associated with an entity.  This is *not* the user ID on a `user` entity; that is defined by the `ENTITY_ID` attribute of that entity.  Rather, it is used for attributes on *other* entities that reference users.  For example, `rating` entities have a `USER_ID` attribute that stores the ID of the user who provided the rating.
- `ITEM_ID`, like `USER_ID` but references an item.
- `RATING`, used to provide the rating value.
- `TIMESTAMP`, a timestamp associated with an entity such as a rating.

## Data Access

[DataAccessObject]: /apidocs/org/lenskit/data/dao/DataAccessObject.html
[query]: /apidocs/org/lenskit/data/dao/DataAccessObject.html#query-org.lenskit.data.entities.EntityType-

Data access in LensKit revolves around the data access object interface, [DataAccessObject][].  This interface provides query access to a database of entities.  It has two primary operations:

- Get the IDs of all entities of a particular type, with `getEntityIds`
- Query the database for entities of a particular type, possibly matching other conditions, with [query][query]

The `query` method exposes a fluent interface for writing and using database queries.  For example, the following query:

~~~java
List<Entity> ratings = dao.query(CommonTypes.RATING)
                          .withAttribute(CommonAttributes.USER_ID, 42L)
                          .get();
~~~

will retrieve all ratings by user 42 as a list.

Other useful methods for queries include:

- `orderBy` specifies a sort order.
- `groupBy` makes the query entities grouped by a `long` attribute.
- `stream` returns data as a closable stream instead of a list; with some data sources, this may avoid slurping the entire query results into memory.  Be sure to close your streams, for example with a `try`-with-resources block, to avoid resource exhaustion.
- `count` counts the results of the query without returning them.

LensKit components depend on the data access object just like they do any other component.

## View Classes

The `Entity` interface is very general, but is not terribly convenient for commonly-used data.  To solve this, LensKit supports *view classes* that are subtypes of `Entity` and provide Java APIs for the attributes generally associated with a particular type of entity and often optimized storage of its data.

[Rating]: /apidocs/org/lenskit/data/ratings/Rating.html

One such class is  [Rating][].  It exposes the common attributes of a rating entity: user and item IDs, rating value, and timestamp.

To use a view class, pass it to the `asType` query method:

~~~java
List<Rating> ratings = dao.query(CommonTypes.RATING)
                          .asType(Rating.class)
                          .withAttribute(CommonAttributes.USER_ID, 42L)
                          .get();
~~~

As a shortcut, when retrieving entities of a view class's default type (and the default entity type for `Rating` is `rating`), you can just write:

~~~java
List<Rating> ratings = dao.query(Rating.class)
                          .withAttribute(CommonAttributes.USER_ID, 42L)
                          .get();
~~~
