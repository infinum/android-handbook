## Views

1. **CircleImageView**

	A fast circular ImageView perfect for profile images. As this is just a custom ImageView and not a custom 	Drawable or a combination of both, it can be used with all kinds of drawables, e.g. a PicassoDrawable from Picasso or other non-standard drawables.

	More info with code examples is available [here](https://github.com/hdodenhof/CircleImageView).

2. **Glide**

	Glide is a fast and efficient open source media management and image loading framework for Android. It wraps media decoding, memory and disk caching, and resource pooling into a simple and easy-to-use interface. It also has support for animated GIFs.

	More info with code examples is available [here](https://github.com/bumptech/glide).

3. **MPAndroidChart**

    MPAndroidChart is a powerful graph view library supporting line, pie, bar radar, bubble graphs and much more. It also includes other features like scaling, panning and animations.

    More info with code examples is available [here](https://github.com/PhilJay/MPAndroidChart).

4. **Lottie**

    Lottie is the go-to library if you need to add some awesome custom animations to your app.
    It is powered by the commonly used JSON format to define animations of all kinds.
    The resource files are generated from Adobe After Effects, and you can find lots of community made animations [here](https://lottiefiles.com/).

    More info on the library (with code examples) is available [here](https://github.com/airbnb/lottie-android).

## Networking

1. **Moshi**

	Just like GSON, Moshi by Square is used to convert Java and Kotlin objects into their JSON representation (and vice versa). Also used commonly with Retrofit and parsing API responses/requests.

	Another reason why we started using Moshi over GSON on some projects is the use of the JSON API specification. You can find [more info here](http://jsonapi.org/).
	Moshi has a great adapter that can easily parse, which makes it an invaluable asset in our library arsenal.

	More info with code examples is available [here](https://github.com/square/moshi).
	More info about Moshi and JSON API is available [here](https://github.com/kamikat/moshi-jsonapi).

2. **Retrofit**

	Retrofit is a type-safe HTTP client for Android and Java. It uses annotations to describe a HTTP request and has support for URL parameter replacement and query parameter. Also, it allows you to upload Multipart request bodies and files.

	More info with code examples is available [here](http://square.github.io/retrofit/).

3. **OkHttp**

	OkHttp is a HTTP & SPDY client for Android and Java applications. It also provides you with MockWebServer, which can be used for testing.

	More info with code examples is available [here](http://square.github.io/okhttp/).

4. **Chucker**

	Chucker is a network library that logs the requests and responses during runtime on the device itself. This library should be enabled on staging and/or debug builds and should **never** be used in production. This is a required library that should always be implemented in the aforementioned environments.

	More info with code examples is available [here](https://github.com/ChuckerTeam/chucker).

## Database

1. **Room**

	Room is a library that provides an abstraction layer over the SQLite database. It supports all SQL functionalities.

	More info with code examples is available [here](https://developer.android.com/topic/libraries/architecture/room.html).

	What's important about this library are migrations. They are crucial if you wish to migrate your database to a new version. If you don't set up migrations, the room will not crash, nor will it log anything. What it will do is **delete all database data and rebuild the database.** You do **not** want this to happen in production, so be careful when using it and set up your migrations properly. If you are not sure how to do it, ask your team lead to help you out.

2. **DbInspector**

	An in-house developed library for viewing the contents of the in-app database for debugging purposes. No need to pull the database from a rooted phone.

	More info with code examples is available [here](https://github.com/infinum/android_dbinspector).

## Testing

1. **Mockito**

	“Mockito is a mocking framework that tastes really good.” We use it to create readable unit tests which produce clean verification errors.

	More info with code examples is available [here](http://mockito.org/).

2. **Mockito Kotlin**

    A small library that introduces Kotlin helpers for working with Mockito. If you use Kotlin when writing tests,
    this will make them shorter and more readable.

    More info with code examples is available [here](https://github.com/nhaarman/mockito-kotlin).

3. **OkHttp MockWebServer**

    It can be very difficult to mock web requests and responses. This is where MockWebServer comes in handy.
    It provides an easy-to-use scriptable web server implementation that can be used for testing your HTTP client.

    More info with code examples is available [here](https://github.com/square/okhttp/tree/master/mockwebserver).

4. **Espresso**

    Espresso is a native Android library for writing UI tests. It can perform actions on your views programmatically
    and check for any output values or states. It also supports some more complex use cases like testing WebViews, Intents and others.

    More info with code examples is available [here](https://developer.android.com/training/testing/espresso).

## Other

1. **Crashlytics**

	Lightweight crash reporting solution with Android Studio support.

	More info with code examples is available [here](https://firebase.google.com/products/crashlytics).

2. **ThreeTen ABP**

	An adaptation of the JSR-310 backport for Android aka a Java 8 Date API backport for Android.

	More info with code examples is available [here](https://github.com/JakeWharton/ThreeTenABP).

3. **Dagger/Hilt**

    Dagger is a dependency injection library for Android, and it is a must-have on every medium to large Android project. It creates global and scoped graphs of depedencies and provides them where needed through injection.
    Hilt is a library built on top of Dagger to simplify the setup process.

    More info on Hilt (recommended) with code examples is available [here](https://developer.android.com/training/dependency-injection/hilt-android).

    For Dagger without Hilt, info and code examples are available [here](https://developer.android.com/training/dependency-injection/dagger-android).

4. **Android KTX**

    Android KTX is a set of Kotlin extensions that are included with Android Jetpack and other Android libraries. KTX extensions provide idiomatic Kotlin to Jetpack, Android platform and other APIs.

    Every project written in Kotlin should use these as they make your code more concise, readable and shorter.

    More info with code examples is available [here](https://developer.android.com/kotlin/ktx).

5. **Lifecycle**

    Lifecycle from AndroidX includes everything you need to make a good MVVM application, such as: ViewModels, LiveData, Lifecycle callbacks, etc.
    It is essential for MVVM arcitecture, but it can also help in lots of other use cases.

    To see all features and variations of androidx.lifecycle, go [here](https://developer.android.com/jetpack/androidx/releases/lifecycle).

    For more info on ViewModel with code examples, go [here](https://developer.android.com/topic/libraries/architecture/viewmodel).

6. **Exoplayer**

    Exoplayer is an all-around solution for media playback. It supports video and audio streaming and local playing.
    Player control UI is also a part of Exoplayer. Since it is well maintained and offers lots of features,
    it should be used when media playback is a requirement.

    More info with code examples is available [here](https://github.com/google/ExoPlayer).

## Useful links

https://android-arsenal.com/
