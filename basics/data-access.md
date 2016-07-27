---
title: Data Access
---

# Connecting to Data

As described in [The LensKit Data Model](data-model.md), LensKit abstracts data access using data access objects (DAOs).  We've seen how to use the DAO to access entities; this page explains how to create and configure a DAO.

Currently, LensKit provides DAOs that support in-memory storage and reading from static files (e.g. CSV files).  The DAO interface can be reimplemented on top of SQL, Hibernate, MongoDB, or whatever your infrastructure requires.

LensKit's data access is all read-only.  LensKit does not provide any facilities for modifying the underlying data; if you need to modify data while LensKit is running, just write the new data directly to the database and provide a LensKit DAO that reads this data.  Prebuilt model components will be out of date with respect to the new data, but many components will take into account the latest data when producing recommendations.

## The Static File Data Source

[StaticDataSource]: /apidocs/org/lenskit/data/dao/file/StaticDataSource.html

The [StaticDataSource][] class compiles a DAO from entities stored in static data files, such as CSV files, and collections in memory.  It can additionally derive entities from mentions in other entity lists, such as synthesizing a set of items from the item IDs in ratings.

Static data sources are described by *data manifests*, YAML files that describe the collection of files comprising the data source.  For example, to read the `ratings.csv` and `movies.csv` files from one of the recent [MovieLens data sets](http://grouplens.org/datasets/movielens), you could use the following:

~~~yaml
ratings:
  file: ratings.csv
  format: csv
  header: true
  entity_type: rating
items:
  file: movies.csv
  format: csv
  header: true
  entity_type: item
  columns: [id, name]
~~~

This describes a data source that reads ratings from `ratings.csv`, using the default column layout for ratings (`user,item,rating,timestamp`), and movie IDs and titles (`name`s) from `movies.csv`.

Once you have such a data source, you can load it:

~~~java
StaticDataSource movielens = StaticDataSource.load(Paths.get("ml-latest/movielens.yml"));
~~~

`StaticDataSource` is a `Provider` of data access objects; call its `get()` method to get a data access object suitable for use in a recommmender.
