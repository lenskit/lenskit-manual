---
title: Evaluator Data Processing
---

# Data Processing in the Evaluator

Before you can run an evaluation, you usually need to perform some pre-processing to the evaluation.
In LensKit 3, we have separated this out from the main evaluator, so that you can use the output
of LensKit's data processing in other tools, or use your own code to prepare your train and test
sets for the evaluator.

## Cross-validating {#crossfold}

Cross-validation is supported in LensKit by first cross-folding a data set to produce a set of
train-test pairs, and then running the [train-test evaluator](../train-test/) on those data
sets.  This is supported by the `crossfold` command and the [Crossfold][] Gradle task.

[Crossfold]: http://localhost:4000/gradle-docs/org/lenskit/gradle/Crossfold.html

Here's a quick example of a crossfold task in a Gradle build script:

~~~groovy
task crossfold(type: Crossfold, group: 'evaluate') {
    input textFile {
        file "data/ml-100k/u.data"
        delimiter "\t"
        // ratings are on a 1-5 scale
        domain {
            minimum 1.0
            maximum 5.0
            precision 1.0
        }
    }
    // test on 5 random ratings from each user
    userPartitionMethod holdout(5, 'random')
    // use 5-fold cross-validation
    partitionCount 5
    // pack data for efficiency
    outputFormat 'PACK'
}
~~~

The options supported by the cross-validation process are defined by [CrossfoldSpec][]; common ones
include:

`partitionCount`
:   The number of train-test data splits to produce.

`name`
:   A name for the data set.

`input`
:   The input data; see [Specifying Input Data](#input-data) for more details.

`outputDir`
:   The output directory; defaults to `${buildDir}/${name}.out`, e.g. `build/crossfold.out`

`outputFormat`
:   The output format; can be one of `CSV`, `CSV_GZ`, `CSV_XZ`, or `PACK`.  If you do not need to
    process the crossfolded output with other software, `PACK` is the most efficient for evaluation.

`method`
:   The cross-folding method.  Can be one of the following:

    PARTITION_USERS
    :   Split the users into `partitionCount` disjoint partitions.  For each partition, produce a
        train-test split by considering some ratings from the users in that partition to be test
        ratings, and the remainder of those users' ratings along with all ratings by other users to
        be the training ratings.

        This is the default option.

    PARTITION_RATINGS
    :   Split the ratings into `partitionCount` disjoint partitions.

    SAMPLE_USERS
    :   Select `partitionCount` disjoint sets of users by random sampling.  Produce train and test
        data as with **PARTITION_USERS**.  This is useful for large data sets where you don't want
        to test on every user.

`sampleSize`
:   When `method` is **SAMPLE_USERS**, determines how many users are used to prepare testing data
    for each train-test set.

`userPartitionMethod`
:   When `method` is **PARTITION_USERS** or **SAMPLE_USERS**, determines how each test users'
    ratings are split into train and test ratings.  Can be one of:

    holdout(*n*, 'random')
    :   Select *n* random ratings to be test ratings, with the remainder used for training.

    holdout(*n*, 'timestamp')
    :   Select the *n* most recent ratings to be test ratings.

    holdoutFraction(*f*, *order*)
    :   Select a fraction *f* (\\(0 < f < 1\\)) of the user's ratings to be test ratings.
        *order* is one of 'random' or 'timestamp', as with **holdout**.

    retain(*n*, *order*)
    :   Select *n* random ratings to be training ratings, with the remainder used for testing.
        *order* is one of 'random' (for random *n* ratings) or 'timestamp' (for the *n* oldest
        ratings).

[CrossfoldSpec]: /apidocs/org/lenskit/specs/eval/CrossfoldSpec.html

## Specifying Input Data {#input-data}

Several tasks take input data from some data set.  While LensKit can take data from a wide variety
of sources by implementing custom data access objects, the evaluator currently supports two types
of data sources.

The [DataSources][] trait provides convenience methods for configuring data sources.  All LensKit
tasks that take data sources as input mix in this trait to provide access to them.

[DataSources]: /gradle-docs/org/lenskit/gradle/traits/DataSources.html

### Text Data

The most common data source is text data, usually in the form of a comma-separated file or similar
delimited column format.  The `textData` method creates such a data set.  It takes the following
options, defined by [TextDataSourceSpec][]:

[TextDataSourceSpec]: /apidocs/org/lenskit/specs/data/TextDataSourceSpec.html
[pd]: /apidocs/org/lenskit/data/ratings/PreferenceDomain.html

`name`
:   A name for the data source.

`file`
:   The file containing the event (rating) data.

`domain`
:   The [*preference domain*][pd], specifying the range and granularity of rating data in the file.
    Takes the sub-commands `minimum`, `maximum`, and `precision`.

`delimiter`
:   The delimiter used to separate fields in the file.

`fields`:
:   A list of fields.  The default is `['user', 'item', 'rating', 'timestamp?']`; the `?` means that
    the timestamp is optional.  The valid fields depend on the type of event builder used.

`builderType`
:   The name of the event builder type to use.  Can be a Java class name, or a shorthand for a
    predefined event type; the default is 'rating'.

`itemFile`
:   A separate file containing a list of all item IDs; useful if there are not ratings for every
    item.

`itemNameFile`
:   A separate comma-separated file associating names or titles with item IDs.

#### MovieLens Data Examples

For example, you can read the MovieLens 100K data set as follows:

~~~groovy
textFile {
    file "ml-100k/u.data"
    delimiter "\t"
    domain {
        minimum 1.0
        maximum 5.0
        precision 1.0
    }
}
~~~

The MovieLens 1M and 10M data sets:

~~~groovy
textFile {
    file "ml-10m/ratings.dat"
    delimiter "::"
    domain {
        minimum 0.5
        maximum 5.0
        precision 0.5
    }
}
~~~

### Pack Data

<div class="alert-box warning">
Pack data is not yet directly supported for manual configuration.
</div>

## Packing Data

Packed data files most often arise from using the crossfolder with `outputFormat 'PACK'`.  However,
if you have produced text data by some other process, you can use the `PackRatings` task to convert
it to LensKit's binary packed rating format:

~~~groovy
task packML10M(type: PackRatings) {
    input textFile {
        file "ml-10m/ratings.dat"
        delimiter "::"
        domain {
            minimum 0.5
            maximum 5.0
            precision 0.5
        }
    }
    output "$buildDir/ml-10m.pack"
}
~~~

The only extra option is `includeTimestamps`: set it to `false` to emit ratings without timestamps.

## Further Reading

-   [Running train-test evaluations](../train-test/)
