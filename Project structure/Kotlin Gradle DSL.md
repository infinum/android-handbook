### Disclaimer: Do not migrate to Kotlin DSL if you have gradle version < 5

Official documentation: https://docs.gradle.org/current/userguide/kotlin_dsl.html  
Detailed migration guide: https://proandroiddev.com/migrating-android-build-scripts-from-groovy-to-kotlin-dsl-f8db79dd6737

## What is Kotlin Gradle DSL ?

Gradle’s Kotlin DSL provides an alternative syntax to the traditional Groovy DSL with an enhanced editing experience in supported IDEs, with superior content assist, refactoring, documentation, and more.

Some of the advantages that you can benefit from using Kotlin Gradle DSL are:  
- Statically typed and type-safe DSL (Domain specific language)  
- Better tooling support (navigation, refactoring, code completion)  
- Interoperability (It runs on JVM)  
- You'll have 100% Kotlin project!


## How does it look like ?

![Kotlin DSL example](/img/kotlinGradleDSL-example.png)

On the first look it doesn't look very different from standard Groovy Gradle file, but you can see that we have "normal" function invocations, value assignment and we are using Kotlin's <code>mapOf</code> in example. Syntax is highligted so you can easily differentiate functions from variables unlike in Groovy. IDE will show you hints about what <code>this</code> is.

## Where to start ?

First of all we want to define file structure that is unique in all projects so we can more easily navigate through gradle files (not that there is too many of them, but it does make life easier). See how it looks in the following image:

![buildSrc](/img/kotlinGradleDSL-buildSrc.png)

So the first step you need to do is to create a package structure like this. Do not add any files there yet. We'll do this step by step.

## Boilerplate files

Some of the files in package structure are always the same. Those are:

1. settings.gradle.kts
2. build.gradle.kts -> You'll see this naming in other gradle files, this one is in the root (buildSrc). It is used to add kotlin dsl dependency into this package.
3. .gitignore (contains .gradle and build generated directories)

Copy them from here and add them into buildSrc directory. Notice that Kotlin Gradle files have .kts extension. 

settings.gradle.kts

```kotlin
// We recommend that you also create a buildSrc/settings.gradle.kts file, which you may leave empty.
// source: https://docs.gradle.org/current/userguide/kotlin_dsl.html#sec:kotlin-dsl_plugin
```

build.gradle.kts

```kotlin
plugins {
    `kotlin-dsl`
}

repositories {
    // The org.jetbrains.kotlin.jvm plugin requires a repository
    // where to download the Kotlin compiler dependencies from.
    jcenter()
}
```

.gitignore

```
.gradle/
build/
```

## Dependencies

Create Dependencies.kt file inside /buildSrc/src/main/kotlin/ directory. You can divide it in three parts: Versions, Dependencies and Plugins. Each of them can be a kotlin object containing constant value of some dependency version or other object which can contain a group of dependencies, like Google or Firebase.

```kotlin
object Versions {
    const val kotlin = "1.3.30"
    const val butterknife = "8.8.1"
    const val glide = "4.7.1"
    const val okHttp = "3.9.0"
    const val couchBase = "1.3.1"
    const val googleSupport = "27.1.1"
    const val annotations = "26.0.0"
    const val gmsVision = "16.2.0"
    const val firebaseCore = "16.0.8"
    const val firebaseMessaging = "17.6.0"
}

object Dependencies {

    object Google {
        const val design = "com.android.support:design:${Versions.googleSupport}"
        const val appcompat = "com.android.support:appcompat-v7:${Versions.googleSupport}"
        const val recycler = "com.android.support:recyclerview-v7:${Versions.googleSupport}"
        const val cardview = "com.android.support:cardview-v7:${Versions.googleSupport}"
        const val annotations = "com.android.support:support-annotations:${Versions.annotations}"
        const val vision = "com.google.android.gms:play-services-vision:${Versions.gmsVision}"
    }

    object Firebase {
        const val core = "com.google.firebase:firebase-core:${Versions.firebaseCore}"
        const val messaging = "com.google.firebase:firebase-messaging:${Versions.firebaseMessaging}"

    }

    object CouchBase {
        const val android = "com.couchbase.lite:couchbase-lite-android:${Versions.couchBase}"
        const val core = "com.couchbase.lite:couchbase-lite-java-core:${Versions.couchBase}+patch-709"
        const val js = "com.couchbase.lite:couchbase-lite-java-javascript:${Versions.couchBase}"
    }

    object Glide {
        const val processor = "com.github.bumptech.glide:compiler:${Versions.glide}"
        const val core = "com.github.bumptech.glide:glide:${Versions.glide}"
    }

    object OkHttp {
        const val interceptor = "com.squareup.okhttp3:logging-interceptor:${Versions.okHttp}"
        const val core = "com.squareup.okhttp3:okhttp:${Versions.okHttp}"
    }

    object ButterKnife {
        const val core = "com.jakewharton:butterknife:${Versions.butterknife}"
        const val processor = "com.jakewharton:butterknife-compiler:${Versions.butterknife}"
    }

    const val kotlin = "org.jetbrains.kotlin:kotlin-stdlib:${Versions.kotlin}"
    const val multidex = "com.android.support:multidex:1.0.1"
    const val material_progress = "com.pnikosis:materialish-progress:1.7"
    const val gson = "com.google.code.gson:gson:2.7"
    const val eventbus = "de.greenrobot:eventbus:2.4.0"
    const val threeten = "com.jakewharton.threetenabp:threetenabp:1.0.5"
    const val commons_codec = "commons-codec:commons-codec:1.9"
    const val sliding_tabs = "com.astuetz:pagerslidingtabstrip:1.0.1@aar"
    const val timber = "com.jakewharton.timber:timber:4.5.1"
    const val fab = "com.github.clans:fab:1.6.4"
    const val crop_view = "com.isseiaoki:simplecropview:1.1.4"
    const val photo_view = "com.github.chrisbanes:PhotoView:1.2.6"
    const val open_tok = "com.opentok.android:opentok-android-sdk:2.13.0"
    const val rxandroid = "io.reactivex.rxjava2:rxandroid:2.0.1"
    const val rxjava = "io.reactivex.rxjava2:rxjava:2.0.8"
    const val exomedia = "com.devbrackets.android:exomedia:4.0.0"
    const val crashlytics = "com.crashlytics.sdk.android:crashlytics:2.4.0@aar"
    const val dbinspector = "im.dino:dbinspector:3.3.0@aar"
    const val junit = "junit:junit:4.12"
    const val mockito = "org.mockito:mockito-core:2.2.9"
    const val goldeneye = "co.infinum:goldeneye:1.1.2"
}

object Plugins {

    const val androidApplication = "com.android.application"
    const val hugo = "com.jakewharton.hugo"
    const val fabric = "io.fabric"
    const val dexcount = "com.getkeepsafe.dexcount"
    const val googleServices = "com.google.gms.google-services"

    object Kotlin {

        const val androidExtensions = "android.extensions"
        const val kapt = "kapt"
        const val android = "android"
    }
}
```

## BuildConfigFields

This file does not have a generic format, it depends on your project. It is used to manage all the properties that your project flavours need. One thing is sure, buildConfigField takes triple containing type, name and value which is perfect job for Kotlin's <code>Triple</code> type. This file is important because it can reduce the size of main gradle file a lot. More the flavours you have, bigger difference you'll see.

## Before getting our hands dirty

For now we have created some packages, added some files, we are getting idea how the things should look. But the most important and tricky part of this migration is application's gradle file. Before getting hands dirty, let's do some preparation inside of our existing gradle file.

1.&nbsp;Update gradle plugins 

You want to have latest version of gradle plugins since maybe there were some improvements or fixes regarding kotlin DSL

2.&nbsp;Disable gradle auto-import

You'll thank me for this later. When migration is in process, if this is on, you'll never get rid of red highlited code no matter if there is an error or not. Gradle will repeatedly try to import some stuff but it wont recognise most of the code so there's a mess... Just disable it.

3.&nbsp;Convert string quotes

Simple but useful tip. Just Replace all the <code>'</code> with <code>"</code>. Groovy supports both, which won't cause any errors and you'll be ready for kotlin strings.

4.&nbsp;Separate function invocation and property assignment

We touched this in introduction and now is the time to fix it. In Groovy you can do this:

<code>applicationId "co.infinum.your.awesome.project"</code>  
<code>classpath "com.android.tools.build:gradle:3.3.0"</code>

So the sytax is the same for both lines, but first line is property assignment and second one is function invocation. What you need to do is recognise which one is it and convert them into this:

<code>applicationId = "co.infinum.your.awesome.project"</code>  
<code>classpath("com.android.tools.build:gradle:3.3.0")</code>

## Getting hands dirty

Ok, we are ready now. Let's do it.

Rename: build.gradle -> build.gradle.kts

This is hardest part of the job and the moment where magic happens:

![Errors](/img/kotlinGradleDSL-allisfine.png)

As you can see this is not a pleasant view. But don't worry, if you followed steps before you are on right track. General idea on how to continue is simple: Fix all the errors and try to sync it! Of course in practice it is a bit different because once you fix an error there is a chance that gradle still does not recognise it or you need to wait for some time or resync. It is common and first time when you do a migration it will take some time. At this point I would like to recommend one hint that should be very helpful and that is commenting and uncommenting code. To be more specific, comment everyting but basic part of the gradle file (with version code, version name etc.). Fix all the errors there (if they exist) and move on to next chunck of code. Keep in mind that usual errors are syntax of lists, maps etc. After each fix do a gradle re-sync. Good luck!

## Conclusion

Doing conversion and using kotlin gradle files after conversion will be noticeably slower but after few runs all that will be left are benefits that Kotlin provides you alonside with modern gradle files with IDE support and other cool stuff.


