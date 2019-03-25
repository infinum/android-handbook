When writing a class, it's natural for it to make use of other objects. These other objects (or services) are [dependencies](http://tutorials.jenkov.com/ood/understanding-dependencies.html#whatis). The simplest way to write code is to simply create and use those other objects. But this means your object has an inflexible relationship with those dependencies: no matter why you are invoking your object, it uses the same dependencies.

A more powerful technique is to be able to create your object and provide it with dependencies to use. In this way, you can create your object with different dependencies at different times, making your object more flexible. This is [dependency injection](https://en.wikipedia.org/wiki/Dependency_injection), with which you "inject" the dependencies into the object.

![Dependency inversion principle](/img/dependecy_inversion_principle.jpg "Example of dependency inversion principle")

Dependency injection for five-year-olds ([source](http://stackoverflow.com/a/1638961/2643666)):

> When you go and get things out of the refrigerator for yourself, you can cause problems. You might leave the door open, you might get something Mommy or Daddy doesn't want you to have. You might even be looking for something we don't even have or which has expired.
> What you should be doing is stating a need, "I need something to drink with lunch," and then we will make sure you have something when you sit down to eat.


## Why use dependency injection

The main advantage of dependency injection is the reduction of boilerplate code in the project since all work to initialize or set up dependencies is handled by a provider component. With dependency injection, code is more readable and improves other code properties that we care about a lot, such as:

  * Flexibility
  * Reusability
  * Testability

Not every problem is solved with dependency injection, and also there are some disadvantages that come with such great power. As we don't care about actual implementation, our code can become difficult to trace (`Protip:` use <kbd>Alt</kbd>+<kbd>Cmd</kbd> to access the implementation file directly in Android Studio).

## Dagger 2

[Dagger](http://google.github.io/dagger/) is a replacement for `Factory` classes that implements the dependency injection design pattern without the burden of writing the boilerplate. It allows you to focus on interesting classes. Declare dependencies, specify how to satisfy them, and ship your app.

The true power of Dagger 2 lies in reflection, `NOT`! What I'm trying to say is that Dagger 2 does not use reflection at all. Without reflection, we can perform graph, configuration and precondition validation at compile time. All of this means that code is now easy to debug, it is fully traceable, and the build will not pass unless all dependencies can be satisfied.

### Include in project

Add the Android apt (Annotation Processing Tool) plugin. The apt plugin assists in working with annotation processors in combination with Android Studio. It has two purposes—it allows us to configure a compile-time-only annotation processor as a dependency, not including the artifact in the final APK or library, and set up the source paths so that the code that is generated from the annotation processor is correctly picked up by Android Studio.

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

For all of this to work properly, we need to bind the request that is demanding some dependency with a supplier that will provide a matching dependency. This is done by adding annotation to instruct a specific action.

`@Inject`—is used to make a request for certain dependencies. You can annotate `Constructor`, `Field` or `Method` to mark dependencies.

`@Provides`—is used as a method annotation. A method with the provides annotation will register its return type as a dependency that it can supply and match with the requested inject type.

`@Module`—is used as an annotation for classes that provide dependencies. All methods that are annotated with the provides annotation have to be inside module classes.

`@Component`—is used to bind all things together. This creates an interface that will provide the injection start point and enable dependency configuration.

`@Subcomponent`—provides scoped injection. It will append its injection to an already existing component and will live as long as its scope lives (e.g., `Activity`, `Session`, `Fragment`). When its scope dies (e.g., `Activity` is destroyed) subcomponent injections will be removed from the graph.

### Example

Almost every app we develop has a matching API, so we need the [Retrofit](http://square.github.io/retrofit/) service. Let's see how we can use ApiService with dependency injection.

1. We need ApiService interface

```java
	public interface ApiService {

	    @GET("/sprite/{id}")
	    public void getPokemon(@Path("id") int pokemonId, Callback<Pokemon> callback);
	}
```

2. We need someone who wants to use this ApiService (request it as a dependency)

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

3. We need a `Module` that will supply necessary dependencies

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

4. We should create an AppComponent which will provide ApiService to all subcomponents which require it

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

5. Finally, we need to bind our subcomponent to the scope in which it is used

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
