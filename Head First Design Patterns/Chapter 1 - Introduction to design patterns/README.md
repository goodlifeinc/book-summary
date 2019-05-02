# Chapter 1: Strategy pattern :hatching_chick:

Consider SimUDuck :baby_chick: applcation - duck pond simulation game. 
:video_game: The game can show a large variety of duck species swimming and making quacking sounds.

1. **Initial program**

    We have a base class `Duck` with only 3 methods in it: `quack()`, `swim()` and `display()`. All other duck types inherit from it. The `display()` method is abstract since all ducks subtypes looks different.

    At the beginning we have only 2 types of ducks: `CityDuck` and `MountainDuck`. Each one of these classes is responsible for implementing its own `display()` behavior.

2. **Expanding the application**

    :question: What will happen if we want to add a new method in the base class - `fly()` and a new kind of duck which is a `RubberDuck`? 

    All subclasses inherit `fly()` including the rubber duck.  
    **A problem occurs** - the rubber duck cannot fly. :boom:

    There is a flaw in the design of our program. By putting `fly()` in the superclass, it gave flying ability to **ALL** ducks, including those that shouldn’t.  
    Also the rubber duck cannot quack, it squeak, so what can we do -> we `override` the quack method. Therefor we can do the same thing with `fly()`. 

    But what will happen if we want to add a new type of duck - `WoodenDuck`? It cannot quack, nor fly?

    **Inheritance was not a good idea** :grey_exclamation:  
    The spec will keep changing and we will be forced to look at and possibly override `fly()` and `quack()` for every new `Duck` subclass that’s ever added to the program... forever.

3. **Using interfaces**

    We need a cleaner way to have some of the ducks *fly* and *quack*.  
    We can remove the two method from the base class and create two new interfaces `Flyable` and `Quackable`. That way only the ducks that are supposed to fly will implement `Flyable` and the ones that are supposed to quack will implement the `Quackable` interface.

    That solves only part of the problem. **Using interfaces completely destroys code reuse for those behaviors, so it just creates a different maintenance nightmare**. Also there might be more than one kind of flying behavior even among the ducks that do fly.

    :exclamation: :pencil: So the first rule when designing an application is to **identify the aspects of your application that vary and separate them from what stays the same.**

    In other words, if you've got some aspect of your code that is changing (every new requirement), then you know you've got a behavior that needs to be pulled out and separated from all the stuff that doesn’t change.

    In our example what we can do is extract the `fly()` and `quack()` methods from the `Duck` class and put them in two separate classes representing duck behaviors. We want to keep things flexible and **change the behavior dynamically.** In other words, we should include behavior setter methods in the `Duck` classes so that we can change for example the `MallardDuck`'s flying behavior at runtime.

    :exclamation: :pencil: Always **program to an interface, not an implementation.**

    - We can use interfaces to represent each behavior - `FlyBehavior` and `QuackBehavior`. 
    - Each implementation of a behavior will implement those interfaces. 
    - Our `Duck` class will have two fields for *Fly-* and *QuackBehavior*.

    This gives us great flexibility. We are creating **has-a** relationship instead of **is-a**. Instead of inheriting their behavior, the ducks get their behavior by being composed with the right behavior object.  
    That let you encapsulate a family of algorithms into their own set of classes, and also lets you change behavior at runtime as long as the object you’re composing with implements the correct behavior interface.

    :exclamation: :pencil: **Favor composition over inheritance.**

:pushpin: **REMEMBER**:
1. Encapsulate what varies.
2. Favor composition over inheritence.
3. Program to interfaces, not implementations.

*Useful resources:*  
:link: [Christopher Okhravi - Strategy pattern](https://www.youtube.com/watch?v=v9ejT8FO-7I&t=)

:link: [Christopher Okhravi - Strategy pattern - Screencast Ep.1](https://www.youtube.com/watch?v=13nftsQUUE0)

:link: [Christopher Okhravi - Strategy pattern - Screencast Ep.2](https://www.youtube.com/watch?v=slfeCvztnT4)

:link: [Sandi Metz - Nothing is Something](https://www.youtube.com/watch?v=OMPfEXIlTVE)

:link: :dollar: [Clean Coders - Episode 27: Strategy & Template Method Patterns by Robert C. Martin](https://cleancoders.com/episode/clean-code-episode-27/show)
