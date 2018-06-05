Gradle is build and dependency system for Android projects since Google introduced Android Studio.

Gradle configuration file is named **build.gradle**.
Each project contains top-level configuration file which looks something like this:

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

Top level file contains global configuration which is applicable to all modules inside your project. (If you don't know what module is, check the following article: [Creating Modules](https://developer.android.com/sdk/installing/create-project.html#CreatingAModule))

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

Let's break down modules' build.gradle into several parts:

###1. Default config
Values defined in `defaultConfig` block override those in AndroidManifest.
`defaultConfig` elements are applied to all build variants, unless that build variant has its own defaultConfig specified.
Example of usual `defaultConfig` block can be seen below.

```gradle
defaultConfig {
    applicationId "co.infinum.appname"
    minSdkVersion 14
    targetSdkVersion 23
    versionCode 3
    versionName "1.1.0"
}
```

###2. Lint options
Lint is a static code analysis tool that checks your source files for potential bugs and optimization improvements. Lint check is mandatory on all projects.

`lintOptions` block defines configuration for Lint. Possible config values can be found on [Lint support](http://tools.android.com/tech-docs/new-build-system/user-guide#TOC-Lint-support) page.

**abortOnError** flag **must be** set to true. You can disable some of the checks after your team leaders' approval. All unapproved disabled checks and *abortOnError false* is subject to yellow card.

```gradle
lintOptions {
    disable 'InvalidPackage', 'MergeRootFrame', 'InconsistentLayout', 'ContentDescription'
}
```

###3. Signing configs
`signingConfigs` can contain one or more configurations to sign your apk. Each signing configuration should have following properties:

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

Based on the example above, you should reference signing configuration block in the buildType block with `signingConfig signingConfigs.release`.

###4. Build Types
The `buildTypes` element controls how to build and package your app. By default, the build system defines two build types: `debug` and `release`. The `debug` build type includes debugging symbols and is signed with the debug key. The `release` build type is not signed by default.

If your app uses feature which key depends on signingKey then you should sign the debug build with your key instead of using debug (default) key to ease the development for other collaborators. This is done in the example below:

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
`minifyEnabled` defines if the build will be run with [proguard](http://developer.android.com/tools/help/proguard.html). All release builds **must be** built with proguard. `proguardFiles` property defines proguard configuration files.

###5. Product flavors
Product flavor define customized version of the app. Project can have multiple flavors (e.g. [paid | free] or same app targeting different API endpoints, etc.).

Following example creates 3 flavors

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

Product flavor objects are of same type as defaultConfig and share same attributes. This means you can override any default value for specific flavor.

###6. Dependencies
Gradle projects can have dependencies on other components. These components can be external binary packages, or other Gradle projects.
To configure a dependency on an external library jar, you need to add a dependency on the compile configuration.

```gradle
dependencies {
    compile files('libs/foo.jar')
    compile 'com.squareup.retrofit:retrofit:1.9.0'
}
```

###7. Build Variants
Each (Build Type, Product Flavor) combination is called **Build Variant**.
Projects with no flavors still have Build Variants, but the single default flavor is used, nameless, making the list of variants same to the list of Build Types.
To build the variant you want, you should select it from AS menu on the left side.

![Build variant menu](/img/build_variant_1.png "Build variant menu")
![Build variant selection](/img/build_variant_2.png "Build variant selection")

### 8. Adding build version to apk file

Android studio sets the name of the apk file based on the app name, build type and flavor. For example, default output looks like this **app-staging-debug.apk**. If needed this can be changed by adding this code snipped to build.gradle file

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
This code snippet iterates through all build variants and renames apk files to **appName-buildType-versionName.apk**. Modify this example based on your needs.

### 9. Variant filtering

In case you don't want all variants to be available for building, you can filter them out as follows:

```gradle
variantFilter { variant ->
    def ignore = variant.buildType.name.equals('staging') && variant.getFlavors().get(1).name.equals('multiDex')
    variant.setIgnore(ignore);
}
```

`variantFilter` block must be declared inside `android` block of your apps `build.gradle` file (not in the top-level one)
