Writing tests is not the most glamorous part of developing an Android application, but it is an invaluable one. When we talk about testing, in general we have something called a testing pyramid and in Android it is not much different. The testing pyramid looks like this:

![testing_pyramid.png](/img/testing_pyramid.png)

The testing pyramid above consists of three main parts:

* Unit tests
* Integration tests
* UI Tests

As one can see, the foundation of the pyramid are Unit tests which are the highest in number in most projects and in this chapter we will concentrate only on Unit tests. 

## Unit tests

Unit tests are tests that are isolated, fast in execution and cheap to write. Unit tests run on a plain JVM, which means they can't contain code which doesn't run on the JVM out-of-the-box. Since Android code (think activities, fragments, etc.) runs on the Android runtime (ART), and not on plain JVM, this means that we can't run Android code in our unit tests. To pull this off, we have to design our codebase in a way that allows us to neatly separate units that we want to test. Sometimes it is very hard to achieve this, but luckily we have some tools in our arsenal that make our life easier, like jUnit and Mockito.

## Anatomy of a unit test

In order to write better tests that are easy to read and give us the exact amount of information that we need to actually understand the test, it is paramount to choose the appropriate naming and structure for our tests. Let's take a look at an example test which neither follows a naming convention nor has a defined structure:

```kotlin
@Test
fun `login`() {
    val username = "Ivica"
    val password = "123456"
    val expectedUser = User("Ivica", Status.REGISTERED)
    whenever(loginInteractor.login(username, password)).thenReturn(LoginResult(expectedUser, "Token"))
    val actualUser = loginUseCase.login(username, password)
    assertEquals(expectedUser, actualUser)
}
```
The above test does not give us any information about what this code is supposed to test and in which scenario it is going to run. If you take a glance at the test structure, it is not so easy to understand what is happening here. Due to the fact that this is a relatively simple example, you might not see the drawbacks immediately. But it is fair to say that a more complex test without proper structure can easily get messy and hard to read.

### Test method naming

The first thing that is wrong in the test above is the method name. It should tell us:

- The name of the method under test.
- The scenario being tested.
- The expected result.

**There are many naming conventions out in the wild and each has its pros and cons. The most important thing is to pick the one which makes the most sense and stick to it across the codebase!**

For the purpose of this example we will use a naming convention that follows this pattern `methodName ... should ... when`. Now let's try to improve the naming of the test in a way that it gives us more information about the scenario that is being tested here. 

```kotlin
@Test
fun `login should return a user object when the login is successful`() { ... }
```
This already looks much better and the most important thing here is that it gives us enough information to understand what scenario is tested without looking at the actual implementation of the test.

### Test method structure

The structure of a test is often overlooked, because we often just think it is not so important. **Having a well defined and consistent structure results in tests which are more readable and understandable, and in the long run, reduces the maintenance cost.**

In our example, we will use the AAA (Arrange-Act-Assert) structure. This essentially means that we divide our tests in 3 section:

* Arrange - this section is used to set up the objects to be tested. You bring the unit under test to a desired state and configure the dependencies.
* Act - in this section we act upon the unit under test. You call one of its methods, pass the dependencies and capture the output value if any.
* Assert - the assert section is used to verify the outcomes of the act with our expectations. 

Let's take a look how our test example looks like after applying the AAA structure:

```kotlin
@Test
fun `login should return a user object when the login is successful`() {
    // Arrange
    val username = "Ivica"
    val password = "123456"
    val expectedUser = User("Ivica", Status.REGISTERED)
    whenever(loginInteractor.login(username, password)).thenReturn(LoginResult(expectedUser, "Token"))
    // Act
    val actualUser = loginUseCase.login(username, password)
    // Assert
    assertEquals(expectedUser, actualUser)
}
```

The whole test is so much nicer now, don't you think? 
As a side note, when writing tests you do not need to write the actual section names, it is ok to just leave an empty space between those sections.

## Writing good unit tests

Writing good tests is usually hard and sometimes very hard. Nevertheless, we will try to define some general guidelines to help us avoid common mistakes and create better tests at the end of the day.

### Use test doubles

As mentioned in the previous section, we often have to arrange some behaviours or objects that we have in production code, which can sometimes be tricky depending on the complexity of a test. **In order to replace some production code for testing purposes, we use a generic term called Test Double.**
There are several types of test doubles, of which most of the time you will probably use the following:

* Mock - An object which we use for verification by checking whether particular methods were called with specific parameters.
* Stub - An object that holds predefined data and uses it to answer calls during tests. It is used when we cannot or do not want to involve objects that would answer with real data or have undesirable side effects.
* Fake - An object that has a working implementation, but not the same as the production one. Usually it takes some shortcut and has a simplified version of the production code. For example, faking a database with an in memory solution in order to make our unit tests faster.

When we talk about mocks and stubs, we do not need to write our own implementation of those test doubles. Luckily, we have a tool called Mockito which enables us to mock and stub behaviours and dependencies with ease. Keep in mind that Mockito does not enable us to create fakes.

Let's take a look at two examples where we use Mockito stubbing and mocking features.

* Using Mockito for mocking

```kotlin  
val authTokenManager = mock(AuthTokenManager::class.java)
verify(authTokenManger).saveAuthToken("Token")
```
With mocks, we are verifying a specific behavior, which can be seen in the above example using the verify method from Mockito.

* Using Mockito for stubbing

```kotlin  
val authTokenManager = mock(AuthTokenManager::class.java)
whenever(authTokenManager.getAuthToken()).thenReturn("Token")
```
With stubs, we define results for a specific method just like it is shown in the example above. Every time the `getAuthToken` method is called, we will get a stubbed result which is in our case `"Token"`.

It is important to know this terminology because it is an essential part of the testing fundamentals and it will definitely help you on your journey to becoming better at writing tests. You will also often find these terms in the documentation of some testing library that you will use.

### Write testable code

This one is easier said than done. It is a good practice to actually think about your tests when writing production code. In general, it is easy to get a picture of how you will test something. If you ever get stuck and you are not sure how to test something that you actually wrote, that it is the right moment to consult a colleague or research the problem in more detail. 

Here are some general guidelines for writing testable code:

* **Use dependancy injection** - you will hear this a lot but one can not stress enough how important this is. This approach allows us to write loosely coupled code which, in turn, enables us to replace production dependencies with appropriate test doubles. As an end result, this allows us to isolate units of work and write good unit tests.
* **Use abstraction** - it is a good rule of thumb to abstract an implementation in case you are not sure if you can easily test it or not. For example, the use of abstraction allows us to hide a concrete class that uses Android specific implementation. Since Android is not available on JVM, it makes sense to hide it behind an abstraction which we can easily mock in our tests. 
* **Avoid static code** - static classes, methods and values should not be your first choice when developing something, because they are hard to mock and they tend to create global state. In general, when dealing with global state it is hard, but not impossible, to create reliable tests. Sometimes you can not avoid using static code, which is totally fine, but you have to know the implications that this code leaves on your tests. More on that in the next section.

### Tests should not depend on order execution

As you might already know, in jUnit each test class can have more than just one test and each time a test is executed the test class is initialised. This behaviour is on purpose and it is something that you want to retain, because you do not want an older state to mess with tests that are yet to be executed. **Remember, unit tests are isolated!**

In general, global mutable state can be tricky to test and therefore you should tend to avoid it. One example that can ruin this behavior is the singleton design pattern. It is important to note that you should not avoid singletons, they still have their usage in some scenarios. What can be dangerous are stateful singletons and this is where you have to be careful. Since the main purpose of a singleton is to have exactly one instance of an object, this means that our tests in a test class will have the exact same instance of that particular object. It is easy to imagine that one test edits a state of a singleton object and the other test that runs afterwards depends on an initial state of the our object and therefore fails. 

**Tests that are dependent on order execution are not reliable tests and because of that they should be avoided.**

### Avoid flaky tests

**When you write tests, you want them to fail or pass consistently no matter how many times you run them.** Flaky tests describe the exact opposite - a scenario where a test could fail or pass for the exact same configuration. Such behavior could be harmful to developers because these tests are non-deterministic and do not always indicate bugs in the code.

Some common cases where you can introduce flaky tests are when dealing with concurrency, caching, randomness, time dependent operations. When dealing with flaky tests, it is important to know how to approach the problem. [This article](https://hackernoon.com/flaky-tests-a-war-that-never-ends-9aa32fdef359) has some very good points in handling flaky tests so I recommend reading it.

One good rule of thumb from the mentioned article: *If you face a flaky test, do not assume that this is a test problem. You should suspect production code first and then the test. Sometimes a flaky test can be flawless and has just revealed a bug in your code.*

### Keep asserts at a minimum per test case

It is easy to get out of control when we assert things in a test case. Sometimes this is fine and unavoidable but it is a good idea to move in a direction where you keep asserts at a minimum per test case. This does not mean that you should avoid multiple asserts at all costs. For example it is absolutely fine 
if you need to test multiple fields of a single operation, multiple independent asserts is the right approach here. **Keep in mind that you want to test a single operation (the act section of your test), not a single result of an operation.** This is also one of the reasons why AAA tests are helping us write better tests, because they force us to think in a way where we have one act per test.

When talking about multiple asserts, there is one caveat that you need to understand and it is the fact that any assert failing early in the test means the later asserts are not run.

### Keep your tests clean

Developers tend to ignore code quality and readability in tests because of the nature of how tests are written, they tend to have a lot of boilerplate code and you cannot do much about it. This is not a reason why you should behave naughty in tests. **Testing code is still part of your codebase that needs to be maintained and therefore it is your responsibility to create testing code that can be easy to read, understand and reuse.** Therefore, it is ok to create helper functions and classes that will help you in writing tests. A good starting point in making the testing code reusable and cleaner is to simplify the arrange section of your tests, where you can usually find a lot of boilerplate in configuring the objects that you will use during the test.
