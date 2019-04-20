# Chapter 1: Strategy pattern 

Consider SimUDuck applcation - duck pond simulation game. The game can show a large variety of duck species swimming and making quacking sounds.

The initial program has a base class `Duck` with only 3 methods in it: `quack()`, `swim()` and `display()`. The `display()` method is abstract since all ducks subtypes looks different. So different types of ducks inherit from the `Duck` class like `MallardDuck` and `RedheadDuck`. Each one of these classes is responsible for implementing its own `display()` behavior.

But what will happen if we want to add a new method in the base class - `fly()` and also a new kind of duck which is a `RubberDuck`. All subclasses inherit `fly()`, so do the rubber duck, but it cannot fly. We can see that there is a big flaw in the design of our program. By putting `fly()` in the superclass, he gave flying ability to ALL ducks, including those that shouldn’t. Also the rubber duck cannot quack, it squeak, so we `override` the quack method. We think that we can do the same thing with `fly()`. But what will happen if we add a new type of duck - `WoodenDuck`. It cannot quack, nor fly.

Probably inheritance was not a good idea. The spec will keep changing and we will be forced to look at and possibly override `fly()` and `quack()` for every new `Duck` subclass that’s ever added to the program... forever. 

We need a cleaner way to have some of the ducks fly and quack. We can use remove the 2 method from the base class and create 2 new interfaces `Flyable` and `Quackable`. That way only the ducks that are supposed to fly will implement `Flyable` and the ones that are supposed to quack will implement the `Quackable` interface.

But that solves only part of the problem. Using interfaces completely destroys code reuse for those behaviors, so it just creates a different maintenance nightmare. And of course there might be more than one kind of flying behavior even among the ducks that do fly.

So the first rule when designing an application is to **identify the aspects of your application that vary and separate them from what stays the same.**

In other words, if you’ve got some aspect of your code that is changing, say with every new requirement, then you know you’ve got a behavior that needs to be pulled out and separated from all the stuff that doesn’t change.

In our example what we can do is extract the `fly()` and `quack()` methods from the `Duck` class and put them in another classes representing duck behaviors. We want to keep things flexible and change the behavior dynamically. In other words, we should include behavior setter methods in the `Duck` classes so that we can, say, change the `MallardDuck`’s flying behavior at runtime.

Always **program to an interface, not an implementation.**

We will use interfaces to represent each behavior - `FlyBehavior` and `QuackBehavior`. Each implementation of a behavior will implement those interfaces. And our `Duck` class will have 2 fields for Fly- and QuackBehavior. Creating our application like that gives us great flexibility. That way we are creating *has-a* relationship instead of *is-a*. Instead of inheriting their behavior, the ducks get their behavior by being composed with the right behavior object.
Not only does it let you encapsulate a family of algorithms into their own set of classes, but it also lets you change behavior at runtime as long as the object you’re composing with implements the correct behavior interface.

**Favor composition over inheritance.**

**REMEMBER**:
1. Encapsulate what varies.
2. Favor composition over inheritence.
3. Program to interfaces, not implementations.

Useful information:

[Christopher Okhravi - Strategy pattern](https://www.youtube.com/watch?v=v9ejT8FO-7I&t=)

[Christopher Okhravi - Strategy pattern - Screencast Ep.1](https://www.youtube.com/watch?v=13nftsQUUE0)

[Christopher Okhravi - Strategy pattern - Screencast Ep.2](https://www.youtube.com/watch?v=slfeCvztnT4)

[Sandi Metz - Nothing is Something](https://www.youtube.com/watch?v=OMPfEXIlTVE)