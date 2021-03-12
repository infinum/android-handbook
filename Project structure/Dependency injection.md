When writing a class, it's natural for it to make use of other objects. These other objects (or services) are [dependencies](http://tutorials.jenkov.com/ood/understanding-dependencies.html#whatis). The simplest way to write code is to create and use those other objects. But this means that your object has an inflexible relationship with those dependencies; no matter why you are invoking your object, it uses the same dependencies.

A more powerful technique is to be able to create your object and provide it with dependencies to use. This way, you can create your object with different dependencies at different times, which makes it more flexible. This is called [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection) because you "inject" the dependencies into the object.

![Dependency inversion principle](/img/dependecy_inversion_principle.jpg "Example of dependency inversion principle")

Dependency injection for five-year-olds ([source](http://stackoverflow.com/a/1638961/2643666)):

> When you go and get things out of the refrigerator for yourself, you can cause problems. You might leave the door open, you might get something Mommy or Daddy doesn't want you to have. You might even be looking for something we don't even have or which has expired.
> What you should be doing is stating a need, "I need something to drink with lunch," and then we will make sure you have something when you sit down to eat.


## Why use dependency injection

The main advantage of dependency injection is the reduction of boilerplate code in the project since all work to initialize or set up dependencies is handled by a provider component. With dependency injection, code is more readable and improves other code properties that we care about a lot, such as:

  * Flexibility
  * Reusability
  * Testability

Not every problem is solved with dependency injection. There are also some disadvantages that come with such great power. As we don't care about the actual implementation, our code can become difficult to trace (`Protip:` use <kbd>Alt</kbd>+<kbd>Cmd</kbd> to access the implementation file directly in Android Studio).


## Dependency injection in Android

When developing Android applications, you will most likely come across two of the most common DI tools: Dagger and Hilt. Well, it's technically one tool, as Hilt is built on top of Dagger. Dagger is a tool that uses series of configuration classes and annotations in order to build a dependency graph for your application.

Once you set it up properly, you can easily inject everything you need, where you need it. Every dependency needs to be provided in one of the modules, before it can be injected. Since Dagger has a steep learning curve, and can take some time to set up, Google came up with Hilt to make things a lot easier.

You can only use one or the other though. So for legacy projects that already have Dagger set up, migrating to Hilt might not be a best idea, especially if the project is big. For the new projects though, you should definitely consider to side with Hilt right from the start.

We wouldn't want to leave you clueless and having to google stuff, so we will cover both Dagger and Hilt in the following sections.


## Hilt

[Hilt](https://developer.android.com/training/dependency-injection/hilt-android) is a tool that helps you set up Dagger fast and easy in your app. After placing a series of annotations in the right places, Hilt will do all the work for you.

That includes:

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
  * `@Binds` - Also used in modules, but the function can be abstract or an interface function. Intead of creating and returning the instance, the function must return dependency type (usually an interface) and take the concrete implementation as a parameter.
  * `@HiltViewModel` - Place above ViewModels to populate the `ViewModelComponent` enabling view models to be injected.


### How it works

In Hilt, everything happens through annotations. As I mentioned before, the first one you need to add is `@HiltAndroidApp` above your application class. This will generate all the components that you will need.

Keep note of the scope of each generated component. For example, if some dependenci is used in ViewModels only, you need to install it in the `ViewModelComponent`.

`@AndroidEntryPoint` is the next annotation that you will probably use the most. Placed above Activity or Fragment, it makes them viable for injection by adding them to Hilt generated builder modules. No further action required.

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

We are using `SingletonComponent` for the first module, because we need remote config in various places, so we are making sure it's available in the global scope. Also, by using `@Singleton` annotation, we are making sure that a single instance will be made and provided in all places.

In the second module we are using `@Binds` annotation to bind repository interfaces to their implementations. Repositories in this case, are only used in `ViewModel's` so we are installing in `ViewModelComponent`.

Let's set up our `ViewModel`

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

Having placed the `@HiltViewModel` above our `ViewModel`, we can now inject dependencies that are in either `SingletonComponent` or `ViewModelComponent` by using `@Inject` annotation on the `constructor` or above fields or methods.

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

Kotlin and Hilt will provide you with a neat feature that enables you to use Kotlin delegates when inject `ViewModel's`.

Hilt makes the built-in delegate `viewModels` use the injected version of the `ViewModel`.

For other types of dependencies, you might have to use `@Inject` above a `field` directly.

This covers most of the typical use-cases of Hilt that you might encounter. However, Hilt has many more different use cases. For more information see the [official guide](https://developer.android.com/training/dependency-injection/hilt-android).

## Dagger 2

[Dagger](http://google.github.io/dagger/) is a replacement for `Factory` classes that implements the dependency injection design pattern without the burden of writing the boilerplate. It allows you to focus on interesting classes. Declare dependencies, specify how to satisfy them, and ship your app.

The true power of Dagger 2 lies in reflection, `NOT`! What I'm trying to say is that Dagger 2 does not use reflection at all. Without reflection, we can perform graph, configuration and precondition validation at compile time. All of this means that code is now easy to debug, it is fully traceable, and the build will not pass unless all dependencies can be satisfied.

### Include in project

Add the Android apt (Annotation Processing Tool) plugin. The apt plugin assists in working with annotation processors in combination with Android Studio. It has two purposes. Firstly, it allows us to configure a compile-time-only annotation processor as a dependency, not including the artifact in the final APK or library. Secondly, it lets us set up source paths so that the code generated from the annotation processor is correctly picked up by Android Studio.

Add the following to your build script:

```groovy
buildscript {
    repositories {
      mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:1.3.1'

        // the latest version of the android-apt plugin
        classpath 'com.neenbedankt.gradle.plugins:android-apt:1.7'
    }
}
```

Add the Dagger library, compiler and javax annotation in your project app `build.gradle`. In the end, it should look something like this:

```groovy
apply plugin: 'com.android.application'
apply plugin: 'com.neenbedankt.android-apt'

android {
    // App setup
    ...
}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:21.0.3'

 	// Dagger dependencies
    compile 'com.google.dagger:dagger:2.0'
    apt 'com.google.dagger:dagger-compiler:2.0'
    compile 'org.glassfish:javax.annotation:10.0-b28'
}
```

### How does it work

For all of this to work properly, we need to bind the request that demands some dependency with a supplier that will provide a matching dependency. This is done by adding annotation to instruct a specific action.

`@Inject`—is used to make a request for certain dependencies. You can annotate `Constructor`, `Field` or `Method` to mark dependencies.

`@Provides`—is used as a method annotation. A method with the @Provides annotation will register its return type as a dependency it can supply and match with the requested inject type.

`@Module`—is used as an annotation for classes that provide dependencies. All methods that are annotated with the @Provides annotation have to be inside module classes.

`@Component`—is used to bind all things together. This creates an interface that will provide the injection start point and enable dependency configuration.

`@Subcomponent`—provides scoped injection. It will append its injection to an already existing component and will live as long as its scope lives (e.g., `Activity`, `Session`, `Fragment`). When its scope dies (e.g., `Activity` is destroyed) subcomponent injections will be removed from the graph.

### Example

Almost every app we develop has a matching API, so we need the [Retrofit](http://square.github.io/retrofit/) service. Let's see how we can use ApiService with dependency injection.

1. We need an ApiService interface.

```java
	public interface ApiService {

	    @GET("/sprite/{id}")
	    public void getPokemon(@Path("id") int pokemonId, Callback<Pokemon> callback);
	}
```

2. We need someone who wants to use this ApiService (request it as a dependency).

```java
	public class PokemonInteractorImpl implements PokemonInteractor {

	    private ApiService apiService;

	    private PokemonListener listener;

	    private Callback<Pokemon> callback = new Callback<Pokemon>() {
	        @Override public void success(Pokemon pokemon, Response response) {
	            listener.onSuccess(pokemon);
	        }

	        @Override public void failure(RetrofitError error) {
	            listener.onFailure(R.string.error_connectivity);
	        }
	    };

	    @Inject
	    public PokemonInteractorImpl(ApiService apiService) {
	    	// Constructor annotated with @Inject demands ApiService object
	    	this.apiService = apiService;
		}

	    @Override public void getPokemon(int pokemonId, PokemonListener listener) {

	        this.listener = listener;
	        apiService.getPokemon(pokemonId, callback);
	    }
	}
```

3. We need a `Module` that will supply the necessary dependencies.

```java
	@Module
	public class ApiModule {

	    private static ApiService apiService;

	    private static Client client;

	    @Provides
	    public ApiService provideApiService() {
	        if (apiService == null) {
	            apiService = createApiService();
	        }
	        return apiService;
	    }

	    private ApiService createApiService() {

	    	if (client == null) {
	            client = new OkClient(new OkHttpClient());
	        }

	        RestAdapter.Builder builder = new RestAdapter.Builder()
	                .setClient(client)
	                .setEndpoint(Endpoints.newFixedEndpoint(BuildConfig.API_URL));

	        RestAdapter restAdapter = builder.build();
	        return restAdapter.create(ApiService.class);
	    }
	}
```

```java
	@Module
	public class PokemonModule {

		PokemonView view;

	    public PokemonModule(PokemonView view) {
	    	this.view = view;
		}

	    @Provides
	    public PokemonView provideView() {
	        return view;
	    }

		/**
 		 * Every parameter passed in the method will first search for other provide methods to satisfy dependency.
 		 * If the first search fails, the parameter will be created using a constructor that is annotated with @Inject.
 		 * If both of these searches fail, this will result in a compile time error.
 		 */
		@Provides
	    public PokemonPresenter providePresenter(PokemonPresenterImpl presenter) {
	        return presenter;
	    }

		@Provides
	    public PokemonInteractor provideInteractor(PokemonInteractorImpl interactor) {
	        return interactor;
	    }
	}
```

4. We should create an AppComponent which will provide ApiService to all subcomponents which require it.

```java
  	@Component(modules = ApiModule.class)
	public interface AppComponent {

		// for each subcomponent whose @Provides require ApiService
		// we need to create a getter
		PokemonComponent plus(PokemonModule module);
	}

	public class PokemonApplication extends Application {

		private static AppComponent appComponent;

		@Override
	    public void onCreate() {
	        super.onCreate();
	        appComponent = DaggerAppComponent.create();
	    }

	    public static AppComponent getAppComponent() {
	    	return appComponent;
		}
	    ...
	}
```

5. Finally, we need to bind our subcomponent to the scope in which it is used.

```java
	/**
	 * Pokemon module is used to provide PokemonPresenter and PokemonInteractor in the spirit of MVP
	 */
	@Subomponent(modules = PokemonModule.class)
	public interface PokemonComponent {

	    void inject(PokemonActivity activity);
	}

	public class PokemonActivity extends Activity implements PokemonView {

		@Inject
		PokemonPresenter pokemonPresenter;

		@Override
	    public void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        setContentView(R.layout.activity_pokemon);

	        PokemonApplication.getAppComponent.plus(new PokemonModule(this)).inject(this);
	    }

	    ...
	}
```

For more detailed examples and better explanation of how Dagger works under the hood, have a look at this [presentation by Jake Wharton](https://speakerdeck.com/jakewharton/dependency-injection-with-dagger-2-devoxx-2014).
