Gradle has been the build and dependency system for Android projects since Google introduced Android Studio.

The Gradle configuration file is named **build.gradle**.
Each project contains a top-level configuration file which looks something like this:

```gradle
buildscript {
    repositories {
        jcenter()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.2.3'
    }
}

allprojects {
    repositories {
        jcenter()
    }
}
```

A top-level file contains a global configuration which is applicable to all modules inside your project. (If you don't know what a module is, check out the following article: [Projects overview](https://developer.android.com/studio/projects#ApplicationModules)).

Your main module (usually named **app**) contains its own **build.gradle**.
Its contents usually look like this:

```gradle
apply plugin: 'com.android.application'

android {
    compileSdkVersion 22
    buildToolsVersion "22.0.1"

    defaultConfig {
        // default config
    }

    lintOptions {
       // lint options here
    }

    signingConfigs {
        release {
            // release config here
        }
    }

    buildTypes {
        // module build types
    }

    flavorDimensions 'api'

    productFlavors {
       // module flavors
    }
}

dependencies {
    // dependencies here
}
```

Let's break down the module's build.gradle into several parts:

### 1. Default config
Values defined in the `defaultConfig` block override those in AndroidManifest.
`defaultConfig` elements are applied to all build variants, unless a build variant has its own defaultConfig specified.
An example of a typical `defaultConfig` block can be seen below.

```gradle
defaultConfig {
    applicationId "co.infinum.appname"
    minSdkVersion 14
    targetSdkVersion 23
    versionCode 3
    versionName "1.1.0"
}
```

### 2. Lint options
Lint is a static code analysis tool that checks your source files for potential bugs and optimization improvements. A lint check is mandatory on all projects.

The `lintOptions` block defines the configuration for lint. For more info, check out the [lint support](https://developer.android.com/studio/write/lint?hl=en) page.

The **abortOnError** flag **must be** set to true. You can disable some of the checks after your team leader's approval. All unapproved disabled checks and *abortOnError false* are subject to a yellow card.

```gradle
lintOptions {
    disable 'InvalidPackage', 'MergeRootFrame', 'InconsistentLayout', 'ContentDescription'
}
```

###3. Signing configs
`signingConfigs` can contain one or more configurations used to sign your APK. Each signing configuration should have the following properties:

* keyAlias
* keyPassword
* storeFile
* storePassword

```gradle
signingConfigs {
    release {
        keyAlias '{aliasvalue}'
        keyPassword '{keyPasswordValue}'
        storeFile file('mykeystore.jks')
        storePassword '{storePasswordValue}'
    }
}
```

Based on the example above, you should reference the signing configuration block in the buildType block with `signingConfig signingConfigs.release`.

###4. Build types
The `buildTypes` element controls how your app is built and packaged. By default, the build system defines two build types: `debug` and `release`. The `debug` build type includes debugging symbols and is signed with the debug key. The `release` build type is not signed by default.

If your app uses a feature whose key depends on signingKey, you should sign the debug build with your key instead of using the debug (default) key to make the development easier for other collaborators. This is done in the example below:

```gradle
 buildTypes {
    debug {
        minifyEnabled false
        debuggable true
        applicationIdSuffix '.dev'
        signingConfig signingConfigs.release
    }

    release {
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        signingConfig signingConfigs.release
    }
}
```
`minifyEnabled` defines if the build will be run with [ProGuard](http://developer.android.com/tools/help/proguard.html). All release builds **must be** built with ProGuard. The `proguardFiles` property defines ProGuard configuration files.

###5. Product flavors
Product flavors define a customized version of the app. A project can have multiple flavors (e.g., [paid | free] or the same app targeting different API endpoints, etc.).

The following example creates three flavors:

```gradle
 flavorDimensions 'api'
 productFlavors {
    dev {
        flavorDimension 'api'
        applicationId 'co.infinum.appname.dev'
    }
    staging {
        flavorDimension 'api'
        applicationId 'co.infinum.appname.staging'
    }
    production {
        flavorDimension 'api'
    }
}
```

Product flavor objects are of the same type as defaultConfig and share the same attributes. This means that you can override any default value for a specific flavor.

###6. Dependencies
Gradle projects can have dependencies on other components. These components can be external binary packages or other Gradle projects.
To configure a dependency on an external library jar, you need to add the dependency on the compile configuration.

```gradle
dependencies {
    compile files('libs/foo.jar')
    compile 'com.squareup.retrofit:retrofit:1.9.0'
}
```

###7. Build variants
Each (build type, product flavor) combination is called a **build variant**.
Projects with no flavors still have build variants, but the single default flavor is used, nameless, making the list of variants the same as the list of build types.
To build the variant you want, you should select it from the AS menu on the left side.

![Build variant menu](/img/build_variant_1.png "Build variant menu")
![Build variant selection](/img/build_variant_2.png "Build variant selection")

### 8. Adding a build version to an APK file

Android Studio sets the name of the APK file based on the app name, build type, and flavor. For example, the default output looks like this: **app-staging-debug.apk**. If necessary, this can be changed by adding the following code snippet to build.gradle file:

```gradle
android.applicationVariants.all { variant ->
    def appName

    if (project.hasProperty("applicationName")) {
        appName = applicationName
    } else {
        appName = parent.name
    }

    variant.outputs.each { output ->
        def newApkName

        if (output.zipAlign) {
            newApkName = "${appName}-${output.baseName}-${variant.versionName}.apk"
        } else {
            newApkName = "${appName}-${output.baseName}-${variant.versionName}-unaligned.apk"
        }
        output.outputFile = new File(output.outputFile.parent, newApkName)
    }
}
```
This code snippet iterates through all build variants and renames APK files to **appName-buildType-versionName.apk**. Modify this example depending on your needs.

### 9. Variant filtering

In case you don't want all variants to be available for building, you can filter them out like this:

```gradle
variantFilter { variant ->
    def ignore = variant.buildType.name.equals('staging') && variant.getFlavors().get(1).name.equals('multiDex')
    variant.setIgnore(ignore);
}
```

The `variantFilter` block has to be declared inside the `android` block of your app's `build.gradle` file (not in the top-level one).
