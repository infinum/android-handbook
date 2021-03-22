## Hilt

[Hilt](https://developer.android.com/training/dependency-injection/hilt-android) is a tool that helps you set up Dagger fast and easy in your app. After placing a series of annotations in the right places, Hilt will do all the work for you.

This includes:

  * Generating components with different scopes to use with your modules - `SingletonComponent`, `ActivityComponent`, `FragmentComponent`, `ViewModelComponent` etc.
  * Built-in `@ActivityContext` and `ApplicationContext` annotations to make context injection easier
  * Populate generated components with annotated Activities, Fragments, Modules, ViewModels etc.

### Setup

First, add the `hilt-android-gradle-plugin` plugin to your project's root `build.gradle` file:

```groovy
buildscript {
    ...
    ext.hilt_version = '2.33-beta'
    dependencies {
        ...
        classpath "com.google.dagger:hilt-android-gradle-plugin:$hilt_version"
    }
}
```

Then, apply the Gradle plugin and add these dependencies in your `app/build.gradle` file:

```groovy
...
apply plugin: 'kotlin-kapt'
apply plugin: 'dagger.hilt.android.plugin'

android {
    ...
}

dependencies {
    implementation "com.google.dagger:hilt-android:$hilt_version"
    kapt "com.google.dagger:hilt-compiler:$hilt_version"
}
```

Hilt uses Java 8 features. To enable Java 8 in your project, add the following to the `app/build.gradle` file:

```groovy
android {
  ...
  compileOptions {
    sourceCompatibility JavaVersion.VERSION_1_8
    targetCompatibility JavaVersion.VERSION_1_8
  }
}
```

### Annotations

Let's start by listing the most important annotations. Later we will expand by explaining how each of them works.

  * `@HiltAndroidApp` - Place above your application class to get the magic going. It generates all previously mentioned components and even some modules that you might need. This one is required for everything to work.
  * `@AndroidEntryPoint` - This one is placed above Activities and Fragments. After doing so, the generated builder modules will be populated with your screens. This is required for each screen that you want to inject to. If you are using ViewModels in your application, this annotation will make sure that the ViewModel can be injected.
  * `@Inject` - Placed above or beside the dependency that you want to inject.
  * `@Module` - Marks a class as a module that provides some dependencies. Each module must also be annotated with `@InstallIn` annotation to specify it's scope.
  * `@InstallIn` - Takes a single parameter that is a component to install the module in. Components have different scopes so make sure that your module is included only where needed
  * `@Provides` - Place above the function in module that returns an instance of the dependency that you want to inject somewhere.
  * `@Binds` - Also used in modules, but the function can be abstract or an interface function. Instead of creating and returning the instance, the function must return a dependency type (usually an interface) and take the concrete implementation as a parameter.
  * `@HiltViewModel` - Place above ViewModels to populate the `ViewModelComponent` enabling view models to be injected.


### How it works

In Hilt, everything happens through annotations. As I mentioned before, the first one you need to add is `@HiltAndroidApp` above your application class. This will generate all the components that you will need.

Keep note of the scope of each generated component. For example, if some dependency is used in ViewModels only, you need to install it in the `ViewModelComponent`.

`@AndroidEntryPoint` is the next annotation that you will probably use the most. Placed above an Activity or Fragment, it makes them viable for injection by adding them to Hilt generated builder modules. No further action required.

`@HiltViewModel` does the same, just for view models. Once you add this annotation, you can feel free to `@Inject` dependencies to your view models. As long as dependencies are provided in some `@Module`.

`@Module` and `@InstallIn` are the pair you will want to use with your modules. Every module consist of either `@Provides` or `@Binds` annotated functions that will provide dependencies as explained earlier.

`@Inject` placed beside a dependency in the `constructor`, `field` or `method` will make sure that your dependency is injected where needed.

Let's dive into a real project example to see how this all works together.

### Example

First we start with an application class:

```kotlin
@HiltAndroidApp
class AliasApp : Application()
```

Then, define some modules that will provide dependencies:

```kotlin
@Module
@InstallIn(SingletonComponent::class)
class RemoteConfigModule {

    @Provides
    @Singleton
    fun remoteConfig(): FirebaseRemoteConfig =
        Firebase.remoteConfig
}
```

```kotlin
@Module
@InstallIn(ViewModelComponent::class)
interface RepositoriesModule {

    @Binds
    fun teamsRepo(teamsRepository: TeamRepositoryImpl): TeamRepository

    @Binds
    fun wordsRepo(wordsRepositoryImpl: WordRepositoryImpl): WordRepository

    @Binds
    fun myWordsRepo(wordsRepositoryImpl: MyWordRepositoryImpl): MyWordRepository

    @Binds
    fun databaseVersionRepo(databaseVersionRepositoryImpl: DatabaseVersionRepositoryImpl): DatabaseVersionRepository
}
```

We are using the `SingletonComponent` for the first module, because we need remote config in various places, so we are making sure it's available in the global scope. Also, by using the `@Singleton` annotation, we are making sure that a single instance will be made and provided in all places.

In the second module, we are using the `@Binds` annotation to bind repository interfaces to their implementations. The listed repositories are only used in `ViewModels` so we are installing them in the `ViewModelComponent`.

Let's set up our `ViewModel`:

```kotlin
@HiltViewModel
class GameViewModel @Inject constructor(
    private val wordRepository: WordRepository,
) : ViewModel() {

   ...

   // Example of usage
   fun onSubmitWord(word: Word) {
       wordRepository.addWord(word)
   }
   ...
}
```

Having placed the `@HiltViewModel` above our `ViewModel`, we can now inject dependencies that are in either `SingletonComponent` or `ViewModelComponent` by using the `@Inject` annotation on the `constructor` or above fields or methods.

In this case, we are injecting the `constructor` with an instance of `WordRepository`.


Now that we have everything in place, let's jump to an activity:

```kotlin
@AndroidEntryPoint
class GameActivity : AppCompatActivity() {

    ...
    private val viewModel by viewModels<GameViewModel>()

    ...

    // Example of usage
    fun onSubmitWord(word: Word) {
        viewModel.onSubmitWord(word)
    }

```

Kotlin and Hilt will provide you with a neat feature that enable you to use Kotlin delegates when inject `ViewModels`.

Hilt makes the built-in delegate `viewModels` use the injected version of the `ViewModel`.

For other types of dependencies, you might have to use `@Inject` above a `field` directly.

This covers most of the typical use-cases of Hilt that you might encounter. However, Hilt has many more different use cases. For more information see the [official guide](https://developer.android.com/training/dependency-injection/hilt-android).
