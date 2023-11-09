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

```kotlin
interface ApiService {

    @GET("/sprite/{id}")
    fun getPokemon(@Path("id") pokemonId: Int, callback: Callback<Pokemon>)
}
```

2. We need someone who wants to use this ApiService (request it as a dependency).

```kotlin
// Constructor annotated with @Inject demands ApiService object
class PokemonInteractorImpl @Inject constructor(private val apiService: ApiService) : PokemonInteractor {
    lateinit var listener: PokemonListener
    private val callback: Callback<Pokemon> = object : Callback<Pokemon>() {
        fun success(pokemon: Pokemon, response: Response) {
            listener.onSuccess(pokemon)
        }

        fun failure(error: RetrofitError?) {
            listener.onFailure(R.string.error_connectivity)
        }
    }

    fun getPokemon(pokemonId: Int, listener: PokemonListener) {
        this.listener = listener
        apiService.getPokemon(pokemonId, callback)
    }
}
```

3. We need a `Module` that will supply the necessary dependencies.

```kotlin
@Module
class ApiModule {

    @Provides
    fun apiService(client: OkHttpClient): ApiService {
        return Retrofit.Builder()
            .baseUrl(BuildConfig.API_URL)
            .client(client)
            .addConverterFactory(MoshiConverterFactory.create())
            .build()
            .create(ApiService::class.java)
    }

    @Provides
    fun okHttpClient(
        logingInterceptor: Interceptor
    ): OkHttpClient {
        return OkHttpClient.Builder()
            .addInterceptor(logingInterceptor)
            .connectTimeout(TIMEOUT_DURATION.first, TIMEOUT_DURATION.second)
            .readTimeout(TIMEOUT_DURATION.first, TIMEOUT_DURATION.second)
            .writeTimeout(TIMEOUT_DURATION.first, TIMEOUT_DURATION.second)
            .build()
    }
}
```

```kotlin
@Module
class PokemonModule(private val view: PokemonView) {

    @Provides
    fun provideView(): PokemonView {
        return view
    }

    /**
     * Every parameter passed in the method will first search for other provide methods to satisfy dependency.
     * If the first search fails, the parameter will be created using a constructor that is annotated with @Inject.
     * If both of these searches fail, this will result in a compile time error.
     */
    @Provides
    fun providePresenter(presenter: PokemonPresenterImpl): PokemonPresenter {
        return presenter
    }

    @Provides
    fun provideInteractor(interactor: PokemonInteractorImpl): PokemonInteractor {
        return interactor
    }
}
```

4. We should create an AppComponent which will provide ApiService to all subcomponents which require it.

```kotlin
@Component(modules = [ApiModule::class])
interface AppComponent {

    // for each subcomponent whose @Provides require ApiService
    // we need to create a getter
    fun plus(module: PokemonModule): PokemonComponent
}

class PokemonApplication : Application {

    private lateinit var appComponent: AppComponent

    @Override
    override fun onCreate() {
        super.onCreate()
        appComponent = DaggerAppComponent.create()
    }

    fun getAppComponent(): AppComponent {
        return appComponent
    }
    ...
}
```

5. Finally, we need to bind our subcomponent to the scope in which it is used.

```kotlin
	/**
	 * Pokemon module is used to provide PokemonPresenter and PokemonInteractor in the spirit of MVP
	 */
@Subcomponent(modules = [PokemonModule::class])
interface PokemonComponent {

    fun inject (activity: PokemonActivity)
}

class PokemonActivity : Activity, PokemonView {

    @Inject
    lateinit var pokemonPresenter: PokemonPresenter

    override fun onCreate(savedInstanceState: Bundle) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_pokemon)

        PokemonApplication.getAppComponent.plus(PokemonModule(this)).inject(this)
    }

    ...
}
```

For more detailed examples and better explanation of how Dagger works under the hood, have a look at this [presentation by Jake Wharton](https://speakerdeck.com/jakewharton/dependency-injection-with-dagger-2-devoxx-2014).
