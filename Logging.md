## Timber

[Timber](https://github.com/JakeWharton/timber) is a library we use for logging. It works on top of Android's normal `Log` class and brings some enhancements. What we can easily do with Timber is to disable logging for release version of our app. Logging is very useful when we are debugging and testing out apps but once app is in production, we don’t want our debug messages to be seen.

Behavior is added to Timber through `Tree` instances. You can install an instance by calling `Timber.plant`. By default are no `Tree` implementations installed, so anything logged by Timber won’t be seen. Installation of `Tree`s should be done as early as possible. The `onCreate()` of your application is the most logical choice.

For debug mode we usually use `DebugTree` implementation which comes with Timber and it prints everything to logcat, just like normal `Log` class. There is also no need to set `TAG` because Timber will automatically figure out from which class it's being called and use that class name as its tag.

To do that simply add this in the `onCreate()` of your app:

```java
if (BuildConfig.DEBUG) {
    Timber.plant(new Timber.DebugTree());
}
```

Now you can call Timber’s static methods to log, just like you would call ones from `Log` class:

```java
Timber.d(...)
Timber.i(...)
Timber.v(...)
Timber.e(...)
Timber.w(...)
Timber.wtf(...)
```

For more information about methods see javadoc [here](http://jakewharton.github.io/timber).

To add Timber to project just add new dependency in `build.gradle`:

```gradle
compile 'com.jakewharton.timber:timber:4.1.0'
```
