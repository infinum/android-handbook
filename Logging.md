## Timber

[Timber](https://github.com/JakeWharton/timber) is a library we use for logging. It works on top of Android's normal `Log` class and brings some enhancements. What we can easily do with Timber is disable logging for the release version of our app. Logging is very useful when we are debugging and testing apps, but once the app is in production, we don’t want our debug messages to be seen.

Behavior is added to Timber through `Tree` instances. You can install an instance by calling `Timber.plant`. No `Tree` implementations are installed by default, so anything logged by Timber won’t be seen. `Tree`s should be installed as early as possible. The `onCreate()` of your application is the most logical choice.

For debug mode, we usually use the `DebugTree` implementation which comes with Timber and it prints everything to logcat, just like normal `Log` class. There is also no need to set `TAG` because Timber will automatically figure out which class it's being called from and use that class name as its tag.

To do that, simply add this in the `onCreate()` of your app:

```java
if (BuildConfig.DEBUG) {
    Timber.plant(new Timber.DebugTree());
}
```

Now you can call Timber’s static methods to log, just like you would call the ones from `Log` class:

```java
Timber.d(...)
Timber.i(...)
Timber.v(...)
Timber.e(...)
Timber.w(...)
Timber.wtf(...)
```

For more information about methods, see a javadoc [here](http://jakewharton.github.io/timber).

To add Timber to a project, just add a new dependency in `build.gradle`:

```gradle
compile 'com.jakewharton.timber:timber:4.1.0'
```
