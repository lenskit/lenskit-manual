# Libraries in Use

LensKit makes use of several libraries throughout its design.  While primary public recommender APIs do not mandate use of these libraries, you will encounter them when you write your own recommender components and algorithms.

## Fastutil

[fastutil]: http://fastutil.di.unimi.it/

We use the [fastutil][] library for optimized primitive collections, such as lists of `long`s or `double`s and maps of the same.  One of the most common types is `Long2DoubleMap`, which we use to represent sparse vectors mapping user or item IDs to values.

[Vectors]: /apidocs/org/lenskit/util/math/Vectors.html
[LongUtils]: /apidocs/org/lenskit/util/collections/LongUtils.html

The LensKit utility class [Vectors][] provides utilities for performing vector computations on vectors represented by `Long2DoubleMap`, such as dot products and Euclidean norms.

We also provide highly memory-efficient, immutable implementations of `LongSortedSet` and `Long2DoubleSortedMap` accessible through the [LongUtils][] utility class (see the `frozenSet` and `frozenMap` methods).  These implementations use sorted arrays to provide $O(lg n)$ lookup and `Vectors` methods take advantage of their layout to provide linear-time pairwise operations such as dot products.  They also do not have excess overhead due to unused space, unlike the hashtable implementations of maps and sets. Further, when none of the `long` values exceed the storage space of an `int`, these implementations will use `int` arrays instead of `long` arrays to further reduce memory consumption.

It is a common pattern to use a `Long2DoubleOpenHashMap` to accumulate results and then convert them to an optimized map using `LongUtils.frozenMap` for longer-term storage.

## Guava

[Guava]: https://github.com/google/guava/wiki

We use [Google Guava][Guava] library extensively in LensKit.  A number of LensKit classes make use of its `Predicate` and `Function` interfaces, and we use its collection and I/O utilities quite a bit.

## Grapht

[Grapht]: http://grapht.grouplens.org/
[Guice]: https://github.com/google/guice

We use the [Grapht][] dependency injection container to configure and instantiate LensKit recommender components.  If you have used [Google Guice][Guice], many aspects of Grapht should be familiar.

## Commons Math 3

[commons-math]: https://commons.apache.org/proper/commons-math/

We use [Apache Commons Math][commons-math] for linear algebra, including dense vectors and matrices.

## Other Libraries

There are several other libraries we use in LensKit, including:

- Commons Lang
- SLF4J
- Joda-Convert
