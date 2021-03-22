## Introduction

When writing a class, it's natural for it to make use of other objects. These other objects (or services) are [dependencies](http://tutorials.jenkov.com/ood/understanding-dependencies.html#whatis). The simplest way to write code is to create and use those other objects. But this means that your object has an inflexible relationship with those dependencies; no matter why you are invoking your object, it uses the same dependencies.

A more powerful technique is to be able to create your object and provide it with dependencies to use. This way, you can create your object with different dependencies at different times, which makes it more flexible. This is called [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection), because you "inject" the dependencies into the object.

![Dependency inversion principle](/img/dependecy_inversion_principle.jpg "Example of dependency inversion principle")

Dependency injection for five-year-olds ([source](http://stackoverflow.com/a/1638961/2643666)):

> When you go and get things out of the refrigerator for yourself, you can cause problems. You might leave the door open, you might get something Mommy or Daddy doesn't want you to have. You might even be looking for something we don't even have or which has expired.
> What you should be doing is stating a need, "I need something to drink with lunch," and then we will make sure you have something when you sit down to eat.


## Why use dependency injection

The main advantage of dependency injection is the reduction of boilerplate code in the project since all work to initialize or set up dependencies is handled by a provider component. With dependency injection, code is more readable and improves other code properties that we care about a lot, such as:

  * Flexibility
  * Reusability
  * Testability

Not every problem is solved with dependency injection. There are also some disadvantages that come with such great power. As we don't care about the actual implementation, our code can become difficult to trace (`Protip:` use <kbd>Alt</kbd>+<kbd>Cmd</kbd> to access the implementation file directly in Android Studio).


## Dependency injection in Android

When developing Android applications, you will most likely come across two of the most common DI tools: Dagger and Hilt. Well, it's technically one tool, as Hilt is built on top of Dagger. Dagger is a tool that uses series of configuration classes and annotations in order to build a dependency graph for your application.

Once you set it up properly, you can easily inject everything you need, where you need it. Every dependency needs to be provided in one of the modules, before it can be injected. Since Dagger has a steep learning curve, and can take some time to set up, Google came up with Hilt to make things a lot easier.

You can only use one or the other though. So for legacy projects that already have Dagger set up, migrating to Hilt might not be the best idea, especially if the project is big. For the new projects though, you should definitely consider to side with Hilt right from the start.

We wouldn't want to leave you clueless and having to google stuff, so we will cover both Dagger and Hilt in the following sections.
