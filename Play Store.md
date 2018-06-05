When releasing an application to Google Play Store there are a few things to go through. A short checklist:

1. Increment versionCode and versionName (compared to the last version on the store)
2. Run tests and make sure all of them are passing
3. Build the signed and obfuscated release version of the application
4. Test the application manually, make sure all API endpoints are pointing to production
5. Upload release .apk to Play Store
6. Add tag for the version on master branch. More about tagging in [Git usage section](/git-usage)

If your app is already in the Play Store, take special care to **test the update** of your app:

1. Install app from play store
2. Use it (log in, fill in-app database with some real data if you can)
3. Send the new apk to your device via mail or use labs to install it (don't use adb because it might remove the old app first and then install the new)
4. Click on the apk on the device to install the apk as an update

This will enable you to find update bugs such as forgetting the users credentials ('remember me' checkbox) and it will also make sure you detect instances where the updated app has a different db schema which makes your app crash after update. In such cases a db migration needs to be prepared for a smooth update.

## Version naming

[Semantic version naming](http://semver.org/) should be used.
In short, a more liberal description:

 - build version should be made of *major*.*minor*.*patch*
 - increase *patch* for bug fixes
 - increase *minor* when adding new functionality
 - increase *major* when removing backwards compatibility or introducing a major change

Side note: builds that are deployed to labs for internal and client testing can have suffixes:

  “-rc{buildNumber}” or “-b{buildNumber}”

The first version of the application that goes in the Play Store should be labeled 1.0.0. An exception to the rule is if the client explicitly requested a different version name. Best effort should be put to explain to the client to use semantic versioning, but in some cases it will not be possible.

## Difference between package name and application id

Every Android application is uniquely identified by the package name. In short, the package name serves two purpose:
 - uniquely identify your app on the Play Store and on the device
 - to specify the package name of the R.java class that is referred to in the code when accessing application resources

Since the gradle build system was introduced those two purposes are distinguished:
 - `applicationId` is used to refer to the final package name that will be used to uniquely identify your application on Play Store
 - package name is used to specify the package for the R.java class

Application id is specified in app module `build.gradle`:

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

Package name is specified in AndroidManifest:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="co.infinum.myapp">
```

For more information check [android documentation](http://tools.android.com/tech-docs/new-build-system/applicationid-vs-packagename).

## Proguard

When building the release version of the application, Proguard must be used to obfuscate source code of the application. Two of the main reasons are decreasing size of the .apk and making it harder for the code to be read after reverse engineering process. For more information on Proguard check [android documentation](http://developer.android.com/tools/help/proguard.html).

In addition to testing the application via unit and instrumentation test, production version of the application must be tested manually as well. One of the reasons is that it is possible to introduce some bugs with Proguard.

Proguard will by default remove classes and methods which seem unused and this can cause problems if some code is used indirectly (for instance via reflection). A common case is when using the GSON library. It relies on reflection to match the JSON key to the class member, so when proguard changes the names of the member variables the JSON will not be properly serialized. To avoid this, always use the `@SerializedName` annotation for field members which need to be deserialized by GSON.

Proguard configurations for most libraries can be found in the respective github `readme.md` files.

## Screenshots

You need to provide at least 3 screenshots for each app which is published to Google Play. Use Clean status bar library (described in [libraries section](/Libraries.md) to make a nice, clean status bar showing only a full battery and clock.
