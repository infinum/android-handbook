## Custom crash handler

By default, your application will be crashed by the OS if an exception is not caught at the moment it is thrown. This results in a system dialog which notifies the user that the app has crashed, and the app is force stopped.

This can be avoided if we define a custom uncaught exception handler:

```java
public class AppCrashHandler implements Thread.UncaughtExceptionHandler {

    private Activity liveActivity;

    public AppCrashHandler(Application application) {
        application.registerActivityLifecycleCallbacks(new Application.ActivityLifecycleCallbacks() {
            @Override
            public void onActivityCreated(Activity activity, Bundle savedInstanceState) {
            }
            @Override
            public void onActivityStarted(Activity activity) {
            }
            @Override
            public void onActivityResumed(Activity activity) {
                liveActivity = activity;
            }
            @Override
            public void onActivityPaused(Activity activity) {
                liveActivity = null;
            }
            @Override
            public void onActivityStopped(Activity activity) {
            }
            @Override
            public void onActivitySaveInstanceState(Activity activity, Bundle outState) {
            }
            @Override
            public void onActivityDestroyed(Activity activity) {
            }
        });
    }
    @Override
    public void uncaughtException(Thread thread, Throwable ex) {
        if(liveActivity != null){
            Intent intent = WelcomeActivity.startFromCrash(liveActivity);
            intent.addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP);
            liveActivity.finish();
            liveActivity.startActivity(intent);
        }

        System.exit(0);
    }
}
```

When an uncaught application exception has been detected by the OS, the *uncaughtException(Thread thread, Throwable ex)* method is called. First, we check if the activity where this exception was caused is still alive. If it is, it means that our app is in the foreground and we have to define a crash fallback. The safest fallback is to simply start the application once again and finish the current activity. If the activity is null, it means that the app is in the background and we can simply force stop the application.

AppCrashHandler has to be registered as the default uncaught exception handler in your application class (in the *onCreate()* method).

```java
  Thread.setDefaultUncaughtExceptionHandler(new AppCrashHandler(this));
```

## Crashlytics

[Crashlytics](https://get.fabric.io/crashlytics), part of [Fabric](https://get.fabric.io), is a crash reporting tool. It provides a simple API for reporting crashes and annotating them with user information and other details. You will be notified of crashes and logged exceptions via email if you decide to use it. More information about the issues is also available, such as the whole stack trace, Android versions that caused the issue, device models, and other details you want to add.

Your team leader can provide you with access to Crashlytics.

To add Crashlytics to a project, you have to edit your `build.gradle`:
```gradle
// top-level build.gradle
buildscript {
    repositories {
        maven {
            url 'https://maven.fabric.io/public'
        }
    }
    dependencies {
        classpath 'io.fabric.tools:gradle:1.+'
    }
}

apply plugin: 'io.fabric'

repositories {
    maven {
        url 'https://maven.fabric.io/public'
    }
}

// app/build.gradle

dependencies {
    compile('com.crashlytics.sdk.android:crashlytics:2.+@aar') {
        transitive = true;
    }
}
```

Then, in the application’s `onCreate()`, add:
```java
Fabric.with(this, new Crashlytics());
```

If you want to, you can also disable Crashlytics reporting in debug mode. Then, instead of the line above, you should use:

```java
CrashlyticsCore crashlyticsCore = new CrashlyticsCore.Builder().disabled(BuildConfig.DEBUG).build();
Fabric.with(this, new Crashlytics.Builder().core(crashlyticsCore).build());
```

At this point, any exception that caused the app to crash is automatically reported to Crashlytics. Also, you can simply use `Crashlytics.logException(e)` and `Crashlytics.log(message)` to log the exception and messages that were caught to Crashlytics. We usually want to have all the crashes and errors from the release version of our app reported to Crashlytics. This way, we can easily see how often our app crashes and why, and what is causing problems to users. On the other hand, while we are using the debug version of our app, we want all those exceptions and messages printed to logcat (and not to Crashlytics). That's what Timber is great for. It enables us to print errors and messages to logcat while we are developing and report those same errors and messages to Crashlytics when the app is in production simply by calling `Timber.e(exception, “Message”)`. Here’s how to do it:

First, we have to define the new `Tree` we will use for the release version. We can use something like this:

```java
private static class CrashReportingTree extends Timber.Tree {

    @Override
    protected void log(int priority, String tag, String message, Throwable t) {
        if (priority == Log.VERBOSE || priority == Log.DEBUG) {
            // avoid reporting
           	return;
        }

        // will write to the crash report but NOT to logcat
        Crashlytics.log(message);

        if (t != null) {
            Crashlytics.logException(t);
        }
    }
}
```


The second thing we have to do is to plant that `Tree` for the release version:

```java
if (BuildConfig.DEBUG) {
    Timber.plant(new Timber.DebugTree());
} else {
    Timber.plant(new CrashReportingTree());
}
```

And that’s all the magic we need for successful logging!

## Crashlytics, remote source control, and CI server

In some cases, we don't want to store Crashlytics *apiKey* and *apiSecret* on a remote source control (i.e., GitHub or Bitbucket). We need to make the following changes to our code in order to properly configure our environment:

1. Remove all Crashlytics meta-tags from AndroidManifest
2. Add *apiKey* and *apiSecret* to the fabric.properties file
3. Add the *fabric.properties* file to .gitignore

If you are using a CI server for tests running and static code analysis, further modifications will be needed. As *fabric.properties* is not included on a remote source control, we need to manually create it after our build has been assembled on the CI server. Paste the following code snippet to your build.gradle file:

```groovy
afterEvaluate {
    initFabricPropertiesIfNeeded()
}

def initFabricPropertiesIfNeeded() {
    def propertiesFile = file('fabric.properties')
    if (!propertiesFile.exists()) {
        if(System.env.crashlyticsApiSecret == null){
            throw new GradleException('Required ORG_GRADLE_PROJECT_crashlyticsApiSecret environment variable not set.')
        }
        else if(System.env.crashlyticsApiKey == null){
            throw new GradleException('Required ORG_GRADLE_PROJECT_crashlyticsApiKey environment variable not set.')
        }
        else{
            def commentMessage = "This is autogenerated fabric property from system environment to prevent key to be committed to source control."
            ant.propertyfile(file: "fabric.properties", comment: commentMessage) {
                entry(key: "apiSecret", value: crashlyticsApiSecret)
                entry(key: "apiKey", value: crashlyticsApiKey)
            }
        }
    }
}

```

We also need to generate two environment variables on our CI server (*ORG\_GRADLE\_PROJECT\_crashlyticsApiKey* and ORG\_GRADLE\_PROJECT\_crashlyticsApiSecret), which will store Crashlytics *apiKey* and *apiSecret*. The variable names have to start with the "ORG\_GRADLE\_PROJECT\_" prefix so that they can be used inside the build.gradle file as environmental variables.


## Useful links

* [Logging with Timber](https://www.youtube.com/watch?v=0BEkVaPlU9A)
