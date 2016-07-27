## Component

A single object, defined by a class typically implementing some interface that defines the component's behavior, that forms a portion of a recommender algorithm.

## Data access object

A class that provides access to data.

## Dependency injection

A design pattern in which classes do not directly instantiate their dependencies; rather, they require their dependencies to be *injected* into them via a constructor parameter or setter.  For example, a component using direct instantiation would construct its dependency as follows:

~~~java
class SomeComponent {
    DepIFace dependency;

    public SomeComponent() {
        dependency = new DepImpl();
    }
}
~~~

Under dependency injection, the dependency comes in from the outside:

~~~java
class SomeComponent {
    DepIFace dependency;

    @Inject
    public SomeComponent(DepIFace dep) {
        dependency = dep;
    }
}
~~~

This pattern helps make classes independent of the implementations of their dependencies, and makes it easier to change implementations because the component does not know what implementation it must use.

LensKit uses a *dependency injector*, [Grapht](http://grapht.grouplens.org), to automatically instantiate algorithm components that are implemented using the dependency injection design pattern.  Grapht scans component constructors for dependencies, instantiates them, and provides them to the constructor.

## Item scorer

[ItemScorer]: /apidocs/org/lenskit/api/ItemScorer.html

An [`ItemScorer`][ItemScorer] computes user-personalized scores for items.  It is at the heart of most LensKit recommender algorithms.
