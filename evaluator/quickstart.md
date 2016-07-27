---
title: Experiment Quickstart
---

# Getting Started with the Evaluator

The LensKit evaluator lets you train algorithms over data sets, measure their
performance, and cross-validate the results for robustness.  This page
describes how to get started using the evaluator for a simple experiment.

LensKit's evaluation capabilities are exposed as a plugin for [Gradle][gradle],
a Java-based tool for automating builds, tests, and other processes.

[gradle]: http://gradle.org

## Prerequisites

We've provided a [template][qs] to help you get started.  This template contains an
example Gradle file, algorithm definitions, and the bootstrap scripts to run Gradle.
You can get the template either by cloning it with Git, or by downloading the source
as a zip archive from GitHub.

[qs]: https://github.com/lenskit/eval-quickstart/tree/lk3

In addition to the template, you will need:

-   Java 7 or later.
-   A tool for analyzing the results.  We provide an example IPython notebook
    that uses Pandas and matplotlib.  If you do not yet have a scientific Python
    environment, [Anaconda Python][conda] is an easy-to-install distribution that
    includes all the necessary pieces.

[conda]: https://www.continuum.io/downloads]

## Project Layout

A LensKit experiment has several files and directories:

`build.gradle`
:   This file controls the entire build and evaluation process.

`algorithms/algorithm.groovy`
:   The configuration file for the algorithm to test.  You'll often
    have more than one of these.

`src/main/java`
:   This directory contains the Java sources for your custom recommender components, just like in
    standard Gradle and Maven projects.

`src/test/java`
:   This directory contains the Java tests for your custom components.

`data`
:   This directory contains the input data.  In the quickstart template,
    the MovieLens data set is automatically downloaded and placed in this
    directory.

`build`
:   This directory is created by the Gradle build process and contains your compiled class files
    and the evaluator's output.

## Running the Evaluation

You can run the quickstart now, by running

    ./gradlew evaluate

If you are on Windows:

    .\gradlew.bat evaluate

This will download the data, compile the custom Java code, and run the
LensKit experiment.

You can then see the results by running

    ipython notebook

which will open a web browser, and you can choose the `analyze-output.ipynb` notebook.  After running the analysis, it should
look something like [this](analysis.html).

## What It Does

This experiment does a 5-fold cross-validation of three algorithms over the
MovieLens 100K data set:

-   Item-item CF
-   User-item average baseline
-   A custom re-implementation of user-item average baseline (to show you
    how to include LensKit code in the experiment project)

The cross-validation is done by partitioning users into 5 sets.  Each set
is used to produce a train-test pair of rating files; the test rating set
contains 20% of each test user's ratings, while the training set contains
the remainder of their ratings along with all the ratings for the non-test
users (the users in the other 4 partitions).

The following metrics are computed:

-   RMSE of rating predictions
-   nDCG of rating predictions (only considers test items, measuring
    the rank effectiveness of the recommender)
-   Mean reciprocal rank with 10-item recommendation lists, considering
    an item ‘relevant’ if the user rated it.

## Gradle Tasks

The example project defines a number of Gradle tasks.  You have already
seen the `evaluate` task.  Some of the other tasks you can run are:

`testClasses`
:   Compile all source code, including unit tests.

`test`
:   Compile the code and run the unit tests for the custom LensKit
    components.

`analyzeResults`
:   Run the evaluation and then process the analysis notebook into a
    static HTML file `build/analysis.html` containing the results of
    the experiment, with charts.

`clean`
:   Delete all compiled code, temporary data files, and evaluation results,
    but *not* the downloaded data files.

`cleanData`
:   Delete the downloaded data files.

Gradle automatically checks the dependencies of each task, and skips it if
nothing affecting its operation has changed since the last time it was run.

## Next

-   [Using the Gradle Plugin](../gradle/)
