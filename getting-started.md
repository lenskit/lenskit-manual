---
title: Getting Started
---

# Getting Started with LensKit

This chapter describes how to embed LensKit in an application.

[lenskit-hello]: http://github.com/lenskit/lenskit-hello

The [lenskit-hello][] project provides a working example of configuring, building, and using a recommender.  This document is based on that code.

## Getting LensKit

We recommend getting LensKit from the Maven central repositories.   To use it from a Gradle project,
add the following to your `build.gradle`:

~~~groovy
repositories {
    mavenCentral()
}
dependencies {
    compile 'org.grouplens.lenskit:lenskit-all:3.0-M1'
}
~~~

Or in Maven:

~~~xml
<dependency>
  <groupId>org.grouplens.lenskit</groupId>
  <artifactId>lenskit-all</artifactId>
  <version>3.0-M1</version>
</dependency>
~~~

`lenskit-all` will pull in all of LensKit except the command line interface.  You can instead depend on the particular pieces of LensKit that you need, if you want.  But `lenskit-all` is a good way to get started.

You can also retrieve LensKit from Maven using SBT, Ivy, or any other Maven-compatible dependency resolver.  If you don't want to let your build system manage your dependencies, download the [binary distribution](http://lenskit.org/download.html) and put the JARs in your project's library directory.

## Configuring the Recommender {#config}

[LenskitConfiguration]: http://lenskit.org/apidocs/org/grouplens/lenskit/core/LenskitConfiguration.html

In order to use LensKit, you first need to configure the LensKit algorithm you want to use.  This consists primarily of selecting the component implementations you want and configuring them with a [LenskitConfiguration][].  For example, to configure a basic item-item kNN recommender with baseline, use the following configuration (save it in a file, such as `item-item.groovy`)

~~~groovy
// Use item-item CF to score items
bind ItemScorer to ItemItemScorer
// let's use personalized mean rating as the baseline/fallback predictor.
// 2-step process:
// First, use the user mean rating as the baseline scorer
bind (BaselineScorer, ItemScorer) to UserMeanItemScorer
// Second, use the item mean rating as the base for user means
bind (UserMeanBaseline, ItemScorer) to ItemMeanRatingItemScorer
// and normalize ratings by baseline prior to computing similarities
bind (UserVectorNormalizer) to BaselineSubtractingUserVectorNormalizer
~~~

You can then load that configuration in your Java program:

~~~java
LenskitConfiguration config = ConfigHelpers.load(new File("item-item.groovy"))
~~~

## Connecting the Data Source

LensKit also requires a data source.  We can use one of the [MovieLens data  sets](http://grouplens.org/datasets/movielens/).  Download the Latest-Small (or Latest) file from there.  You will also need a *data manifest* to tell LensKit how to use it; download [this one](movielens.yml) and save it in the MovieLens data directory (alongside the `.csv` files).

You can then load the data source:

~~~java
StaticDataSource source = StaticDataSource.load("ml-latest-small/movielens.yml");
~~~

## Creating the Recommender

You then need to create a recommender to actually be able to recommend:

~~~java
LenskitRecommender rec = LenskitRecommender.build(config, source);
~~~

When you're finished with a `LenskitRecommender`, close it with `rec.close()`.  `try`-with-resources blocks work great for this:

~~~java
try (LenskitRecommender rec = LenskitRecommender.build(config)) {
    /* do things */
}
~~~

## Generating Recommendations

The recommender object provides access to components like `ItemRecommender` that can do the actual recommendation.  For example, to generate 10 recommendations for user 42:

~~~java
ItemRecommender irec = rec.getItemRecommender();
ResultList recommendations = irec.recommend(42, 10);
~~~

Since we did not configure an `ItemRecommender` when configuring LensKit, it uses the default: the `TopNItemRecommender`, which scores items using the configured `ItemScorer` and returns the *N* highest-scored items.  Since we are using item-item CF, these scores are the raw predicted ratings from item-item collaborative filtering.

You can also also predict ratings with the `RatingPredictor`:

~~~java
RatingPredictor pred = rec.getRatingPredictor();
Result score = pred.predict(42, 17);
~~~

## Next Steps

This is just the beginning! There's a lot more to explore in LensKit, including [how to configure different recommenders](configuration.md), [connecting to data sources](data-access.md), and [evaluating recommenders](../evaluator/quickstart.md).
