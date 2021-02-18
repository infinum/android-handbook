
Often, there will be problems experienced by more than one person in the team or some code which is shared between multiple projects. Those present an excellent opportunity to form a common solution into an open-source library, helping anyone that could make use of it, inside or outside of the company. When creating a library, we strive to make it straightforward, easy-to-use and applicable to everyone, rather than limiting it to our use-cases. 

Developing a library is like working on your mini-project. It can provide you with a sense of pride and accomplishment, as well as being a great learning opportunity.

### [Prince of Versions](https://github.com/infinum/Android-Prince-of-Versions)
Prince of Versions is a library for handling application updates. We use it on almost every project as it solves the issue of handling optional and mandatory updates. 
It is split into two modules: 

* **Queen of Versions** - checks update availability using [in-app updates](https://developer.android.com/guide/playcore/in-app-updates)

* **Prince of Versions** - checks for updates via a remote or local source (usually a JSON file on the server).

### [Sentinel](https://github.com/infinum/android-sentinel)

Sentinel is a simple one screen UI which provides a standardised entry point for tools used in development and QA alongside device, application and permissions data. Including it in your project has a good chance of making your QA very happy, as well as making debugging easier, especially with networking, database, or analytics issues.

### [DbInspector](https://github.com/infinum/android_dbinspector)

DbInspector is a library for viewing the contents of the in-app database for debugging purposes. While somewhat replaced by the new [Android Studio's Database Inspector feature](https://developer.android.com/studio/preview/features#database-inspector), it is useful for QA and is included as a Sentinel tool.

### [Retromock](https://github.com/infinum/Retromock)

Retromock is a library for mocking responses in a Retrofit service. It uses annotations and is very simple and intuitive to implement. If you work for a project with an inconsistent API uptime or are ahead of the backend team in the development, you will undoubtedly find it useful. 

### [Complexify](https://github.com/infinum/android-complexify)

Complexify provides a simple API for checking the quality of a password. You can configure how strong the password has to be, the minimum amount of characters, and ban the most commonly used passwords (or use your own ban list).

### [ConnectionBuddy](https://github.com/zplesac/android_connectionbuddy)

ConnectionBuddy is a utility library for handling connectivity change events. It provides a simple way of listening to connectivity events, particularly useful for applications which need to change behaviour dynamically based on the connection state.

### [Thrifty Retrofit converter](https://github.com/infinum/thrifty-retrofit-converter)

Thrifty Retrofit converter is a [Retrofit](https://github.com/square/retrofit) converter which uses [Thrifty](https://github.com/Microsoft/thrifty) for (de)serialization of Apache Thrift requests and responses.

### [Goldfinger](https://github.com/infinum/Android-Goldfinger)

Goldfinger is a library which simplifies biometric authentication implementation - our go-to on projects which use biometrics.

### [GoldenEye](https://github.com/infinum/Android-GoldenEye)

GoldenEye is a wrapper for Camera1 and Camera2 API which exposes simple to use interface. Android camera is infamously difficult to use, and GoldenEye is our take at simplifying it.
