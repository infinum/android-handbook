## Views

1. **Butterknife**

	Butterknife is a view "injection" library that reduces boilerplate code which you have to write to cast the corresponding views in your layout. This library is a must for all our projects.

	More info with code examples is available [here](http://jakewharton.github.io/butterknife/).

2. **ViewPagerIndicator**

	ViewPagerIndicator provides us with paging indicator widgets compatible with the ViewPager. It helps you provide a clear indicator that additional content exist.

	More info with code examples is available [here](https://github.com/JakeWharton/ViewPagerIndicator).

3. **CircleImageView**

	A fast circular ImageView perfect for profile images. As this is just a custom ImageView and not a custom 	Drawable, or a combination of both, it can be used with all kinds of drawables, i.e., a PicassoDrawable 	from Picasso or other non-standard drawables.

	More info with code examples is available [here](https://github.com/hdodenhof/CircleImageView).

4. **Android Crop**

	Android Crop provides a simple image cropping Activity, based on the code from AOSP. It's backwards compatible with Gingerbread and has nice and clean UI.

	More info with code examples is available [here](https://github.com/jdamcd/android-crop).

5. **Holo Graph Library**

	Slick, easy-to-use graphs with click listeners.

	More info with code examples is available [here](https://github.com/Androguide/HoloGraphLibrary).

6. **Glide**

	Glide is a fast and efficient open source media management and image loading framework for Android, that wraps media decoding, memory and disk caching, and resource pooling into a simple and easy-to-use interface. It also has support for animated GIFs.

	More info with code examples is available [here](https://github.com/bumptech/glide).

7. **Android Support Library package**

	The Support Library package is a set of official Google libraries used for backwards compatibility with older APIs. Each of these libraries supports a specific range of Android platform versions and set of features.

	More info and specific documentation about the Support Library package is available [here] (https://developer.android.com/tools/support-library/features.html).

## Networking

1. **GSON**

	GSON is used for converting Java and Kotlin objects into their JSON representation (and vice versa). Most commonly, we use it in combination with Retrofit to parse API responses.

	More info with code examples is available [here](https://github.com/google/gson).

2. **Moshi**

	Just like GSON, Moshi by Square is used to convert Java and Kotlin objects into their JSON representation (and vice versa). Also used commonly with Retrofit and parsing API responses/requests.

	Another reason why we started using Moshi over GSON on some projects is the use of the JSON API specification, [more info here](http://jsonapi.org/).
	Moshi has a great adapter that can easily parse, which makes it an invaluable asset in our library arsenal.

	More info with code examples is available [here](https://github.com/square/moshi).
	More info about Moshi and JSON API is available [here](https://github.com/kamikat/moshi-jsonapi).

3. **Retrofit**

	Retrofit is a type-safe HTTP client for Android and Java. It uses annotations to describe the HTTP request, and has support for URL parameter replacement and query parameter. Also, it allows you to upload Multipart request bodies and files.

	More info with code examples is available [here](http://square.github.io/retrofit/).

4. **OkHttp**

	OkHttp is an HTTP & SPDY client for Android and Java applications. It also provides you with MockWebServer, which can be used for testing.

	More info with code examples is available [here](http://square.github.io/okhttp/).

5. **Chuck**

	Chuck is a network library that logs the requests and responses in runtime on the device itself. This library should be enabled on staging and/or debug builds and should **never** be used in production. This is a required library that should always be implemented in the aforementioned environments. 

	More info with code examples is available [here](https://github.com/jgilfelt/chuck).

## Database

1. **DBFlow**

	DBFlow is an ORM Android database library with annotation processing. It is built on top of the SQLite database, and has support for all SQL actions and transactions.

	More info with code examples is available [here](https://github.com/Raizlabs/DBFlow).

2. **Room**

	Room is a library that provides an abstraction layer over the SQLite database. It supports all SQL functionalities.

	More info with code examples is available [here](https://developer.android.com/topic/libraries/architecture/room.html).

	What's important about this library are migrations. They are a vital piece if you wish to migrate your database to a new version. If you don't set up migrations, the room will not crash, nor will it log anything. What it will do is **delete all of the database data and rebuild the database.** You do **not** want this to happen in production, so be careful when using it and set up your migrations properly. If you're not sure how to do it, ask your team lead to help you out.

3. **DbInspector**

	An in-house developed library for viewing the contents of the in-app database for debugging purposes. No need to pull the database from a rooted phone.

	More info with code examples is available [here](https://github.com/infinum/android_dbinspector).

## Testing

1. **Mockito**

	“Mockito is a mocking framework that tastes really good.” We use it to create readable unit tests which produce clean verification errors.

	More info with code examples is available [here](http://mockito.org/).

2. **AssertJ Android**

	A set of AssertJ assertions geared toward testing Android.

	More info with code examples is available [here](https://github.com/square/assertj-android).

## Other

1. **Crashlytics**

	Lightweight crash reporting solution with Android Studio support.

	More info with code examples is available [here](https://www.crashlytics.com/).

2. **EventBus**

	EventBus is a publish/subscribe event bus optimized for Android. It simplifies communication between Activities, Fragments, Threads, Services, etc.

	More info with code examples is available [here](https://github.com/greenrobot/EventBus).

	Though it's in some of the projects' codebases, we tend to **avoid** this library as much as possible.

3. **ThreeTen ABP**

	An adaptation of the JSR-310 backport for Android aka a Java 8 Date API backport for Android.

	More info with code examples is available [here](https://github.com/JakeWharton/ThreeTenABP).

4. **Better pickers**

	DialogFragments modeled after the AOSP Clock and Calendar apps to improve UX for picking time, date, numbers, and other things.

	More info with code examples is available [here](https://github.com/derekbrameyer/android-betterpickers).

5. **Clean status bar**

	A Utility library for making Google Play screenshots. It draws over your status bar, showing only a full battery and clock. Use this to have nice, clean screenshots without those pesky empty batteries and notifications.

	More info with code examples is available [here](https://github.com/emmaguy/clean-status-bar).

## Useful links

https://android-arsenal.com/
