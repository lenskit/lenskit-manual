---
title: Evaluator Metrics
---

# Evaluator Metrics

The *predict* and *recommend* tasks provided by the [evaluator](../train-test/) each support
a variety of recommender systems metrics.

To use one of these metrics, just mention its name in the `predict` or `recommend` block of your
`TrainTest` task:

~~~
metric 'rmse'
~~~

Some metrics take additional parameters.  These can be specified with a block:

~~~
metric('ndcg') {
    discount 'exp(5)'
    columnName 'HalfLifeUtility'
}
~~~

## Prediction Accuracy Metrics {#predict}

[predict]: /apidocs/org/lenskit/eval/traintest/predict/package-summary.html

LensKit provides several metrics for measuring prediction accuracy.  The metrics included with LensKit are:

coverage
:   Measures basic coverage statistics: the number of attempted predictions, successful predictions,
    and coverage (successful / attempted).  Produces the output columns `NUsers`, `NAttempted`,
    `NGood`, and `Coverage`.

rmse
:   Measures the root mean squared error of the rating predictions.  Produces user-level column
    `RMSE`, and aggregate-level columns `RMSE.ByUser` (averaging the per-user RMSE) and
    `RMSE.ByRating` (computing the RMSE over all test ratings).

mae
:   Just like **rmse**, except it computes the mean absolute error.

ndcg
:   Measures normalized discounted cumulative gain of the test items ranked by prediction, using
    their ratings (from the test set) as their utilities.  This turns nDCG into a rank effectiveness
    measure.  It takes two configuration parameters:

    discount
    :   The discounting function to apply.  Can be `log2` for base-2 log (the default), `log(n)`
        for base-n log, or `exp(α)` for half-life discounting with a half-life of α[^breese].

    columnName
    :   A name to use for the output column instead of ‘Predict.nDCG’.

[^breese]: John S. Breese, David Heckerman, and Carl Kadie. 1998. [Empirical analysis of predictive algorithms for collaborative filtering](http://dl.acm.org/citation.cfm?id=2074100). In Proceedings of the Fourteenth Conference on Uncertainty in Artificial Intelligence (UAI'98).

These metrics are implemented by classes in the [`org.lenskit.eval.traintest.predict`][predict] package.

[predict]: /apidocs/org/lenskit/eval/traintest/predict/package-summary.html

## Top-N metrics {#topn}

The top-*N* metrics that can be used with the `recommend` task are:

length
:   Measures the length of recommendation lists;  can be used to compute recommendation coverage.
    Produces the column 'TopN.ActualLength'.

ndcg
:   Normalized discounted cumulative gain, applied to top-*N* lists.  Takes the same options as the
    **ndcg** predict metric.  Uses rating values as the utility function.  The default column name
    is 'TopN.nDCG'.

mrr
:   Mean reciprocal rank. Produces the aggregate columns 'MRR' (MRR averaged over all users,
    counting users for whom there were no relevant recommendations as having a reciprocal rank
    of 0) and 'MRR.OfGood' (MRR averaged over all users for whom there was at least one relevant
    item), and the user-level columns 'Rank' and 'RecipRank'.

    This metric takes two parameters:

    goodItems
    :   An item selector (like for `candidateItems` and `excludeItems`) that picks the items that
        will be considered *relevant* for the purposes of finding the first relevant item.  Defaults
        to all test items.

    suffix
    :   A suffix that will be applied to the metric's output columns.  Use this if you want to use
        multiple MRR metrics with different concepts of 'good' in the same evaluation.  If you have
        a suffix of 'AllRated' and a recommend task prefix of 'Size10', the final output file will
        have a column labeled 'Size10.MRR.AllRated'.

map
:   Mean average precision.  Works just like **mrr**.  Produces the global columns 'MAP' and
    'MAP.OfGood' and user-level column 'AvgPrec'.

pr
:   Precision and recall.  Works just like **mrr**.  Produces the columns 'Precision', 'Recall',
    and 'F1' at the user and aggregate levels.

These are implemented by classes in the [`org.lenskit.eval.traintest.recommend`][recommend] package.

[recommend]: /apidocs/org/lenskit/eval/recommend/package-summary.html

## Writing Your Own Metrics

You can write your own metrics by subclassing the [PredictMetric][] or [TopNMetric][] classes.
Each metric should define two or three inner classes:

[PredictMetric]: /apidocs/org/lenskit/eval/predict/PredictMetric.html
[TopNMetric]: /apidocs/org/lenskit/eval/recommend/TopNMetric.html

-   A *context* class, used to aggregate measurements across users.  This class is provided as a
    type parameter to the base class.
-   *Result* classes, extending from `TypedMetricResult`, to describe per-user and aggregate
    results.  These classes should have getters annotated with `@MetricColumn`, providing names
    to be included in the output file.

The constructor should pass the result classes to the superclass constructor to compute the output
columns.

The class should then implement the following methods:

createContext
:   This method receives an algorithm and data set and creates a context object for accumulating
    aggregate measurements for that experimental condition.

getAggregateMeasurements
:   This method takes the context and computes the aggregate results (e.g. average metric value
    over all users).

measureUser
:   This method receives a user and their recommendations or predictions and is responsible for
    measuring the predictions, placing any necessary information into the context, and returning
    the per-user results for that class.

See [RMSEPredictMetric] for an example of how to implement your metric.

[RMSEPredictMetric]: https://github.com/lenskit/lenskit/blob/master/lenskit-eval/src/main/java/org/lenskit/eval/traintest/predict/RMSEPredictMetric.java

Once you have implemented your metric, you can use it by giving its class name to a `metric`
directive, e.g.:

~~~
metric 'org.myorg.metrics.MyCustomMetric'
~~~

You can also register a short name for your metric by providing a file
`META-INF/lenskit/predict-metrics.properties` or
`META-INF/lenskit/topn-metrics.properties`.  This file should map short names to
full class names, e.g.

~~~ini
rmse=org.lenskit.eval.traintest.predict.RMSEPredictMetric
~~~
