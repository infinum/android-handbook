When releasing an application to the Google Play Store, you should make sure you've done everything that's necessary before the release. Here's a short checklist:

1. Increment versionCode and versionName (compared to the last version on the store)
2. Run tests and make sure all of them are passing
3. Build the signed and obfuscated release version of the application
4. Test the application manually; make sure all API endpoints are pointing to production
5. Upload the release .apk to the Play Store
6. Add a tag for the version on `main`/`release` branch. Find out more about tagging in the [Git usage section](/books/android/building-quality-apps/using-git)

If your app is already on the Play Store, take special care to **test the update** of your app:

1. Install the app from the Play Store
2. Use it (log in, fill the in-app database with some real data if you can)
3. Send the new APK to your device by mail or use labs to install it (don't use adb because it might remove the old app first and then install the new one)
4. Click on the APK on the device to install the APK as an update

This will enable you to find update bugs, such as forgetting the user's credentials ('remember me' checkbox), and it will also ensure that you detect instances where the updated app has a different db schema, which makes your app crash after an update. In such cases, a db migration has to be prepared for a smooth update.

## Version naming

[Semantic version naming](http://semver.org/) should be used.
In short, a more liberal description:

 - the build version should be made of *major*.*minor*.*patch*
 - increase *patch* for bug fixes
 - increase *minor* when adding a new functionality
 - increase *major* when removing backwards compatibility or introducing a major change

Side note: builds that are deployed to labs for internal and client testing can have suffixes:

  “-rc{buildNumber}” or “-b{buildNumber}”

The first version of the application that goes on the Play Store should be labeled 1.0.0. An exception to the rule is if the client explicitly requests a different version name. Maximum effort should be made to explain to the client the use of semantic versioning, but in some cases it will not be possible.

## The difference between package name and application ID

Every Android application is uniquely identified by the package name. In short, the package name serves two purposes:
 - to uniquely identify your app on the Play Store and on the device
 - to specify the package name of the R.java class that is referred to in the code when accessing application resources

Since the Gradle build system has been introduced, those two purposes are distinguished:
 - `applicationId` is used to refer to the final package name that will be used to uniquely identify your application on the Play Store
 - the package name is used to specify the package for the R.java class

The application ID is specified in the `build.gradle` app module:

```gradle
apply plugin: 'com.android.application'

android {
    compileSdkVersion 25
    buildToolsVersion '25.0.2'

    defaultConfig {
        applicationId 'co.infinum.myapp'
        minSdkVersion 16
        targetSdkVersion 25
        versionCode 1
        versionName '1.0.0'
    }
    // ...
```

The package name is specified in AndroidManifest:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="co.infinum.myapp">
```

For more information, check out [Android documentation](https://developer.android.com/studio/build/application-id.html).

## ProGuard

When building the release version of the application, ProGuard has to be used to obfuscate the source code of the application. Two main reasons for this are decreasing the size of the .apk and making it harder for the code to be read after the reverse engineering process. For more information on ProGuard, check out [Android documentation](http://developer.android.com/tools/help/proguard.html).

In addition to testing the application through unit and instrumentation tests, the production version of the application has to be tested manually as well. One of the reasons is that it is possible to introduce some bugs with ProGuard.

ProGuard will, by default, remove classes and methods that seem unused, and this can cause problems if some code is used indirectly (for instance, via reflection). A common case is when using the GSON library. It relies on reflection to match the JSON key to the class member, so when ProGuard changes the names of the member variables, JSON will not be properly serialized. To avoid this, always use the `@SerializedName` annotation for field members which need to be deserialized by GSON.

ProGuard configurations for most libraries can be found in the respective GitHub `readme.md` files.

## Screenshots

You need to provide at least three screenshots for each app that is published to Google Play. Use the Clean Status Bar library (described in the [Libraries section](/books/android/useful-tools-and-utilities/libraries)) to make a nice, clean status bar showing only a full battery and clock.
