---
title: Documentation
layout: default
sidenav: doc-nav.html
priority: 1
---

# LensKit Documentation

[wiki]: https://github.com/grouplens/lenskit/wiki/

If you're just getting started with LensKit, start with the following:

- [Getting Started](getting-started.md) to use LensKit in a project
- [Experiment Quickstart](evaluator/quickstart.md) to run recommender experiments with LensKit

Then you will probably want to read the rest of our documentation:

- [LensKit Basics](basics/) — how to use LensKit and work with its interfaces and data structures.
{% if false %}
- [LensKit Algorithms](algorithms/) — more detailed discussions of the algorithms LensKit provides.
- [The LensKit Evaluator](evaluator/) — evaluating recommendation quality.
- [Command Line Tools](cli/) — evaluating recommendation quality.
{% endif %}
- [Java API documentation](/apidocs/)
- The [LensKit wiki](http://github.com/lenskit/lenskit/wiki) has documentation
  on developing LensKit

Additionally, Chapter 3 of [Michael Ekstrand's
dissertation](http://elehack.net/research/thesis/) describes the design of
LensKit's, along with the motivations behind many of the design decisions.  It
is recommended reading for people wanting to work on the LensKit code.
