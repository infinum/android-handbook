Android requires that all APKs be digitally signed with a certificate before they can be installed.
You can find more details in the [official docs](https://developer.android.com/studio/publish/app-signing.html).

Once an app signed with a certificate has been uploaded to the Play Store, **all updates to that app must be signed with the same certificate**.

The signing certificate is stored in a binary file known as the **keystore**.
The keystore is protected with a password and can store multiple certificates, each one protected with another password.

So, to upload an update to the app you need:

- access to the Play Store publishing account
- the keystore that contains the certificate
- the keystore password and the certificate password

If a malicious user obtains all of these, they can upload whatever code they want as an app update to unsuspecting users, so it's very important to restrict access to this information.

It is also very important not to lose the keystore or passwords because we then lose the ability to update the app.

To do that, we use two separate keystores for each project:

- one for development and staging builds
- one for production builds that will be uploaded to the Play Store

## Development keystore

The development keystore should be generated using random alphanumeric passwords. You can do it via the Android Studio GUI or by using [keytool](http://docs.oracle.com/javase/6/docs/technotes/tools/solaris/keytool.html):

```shell
keytool -genkey -v -keystore development.jks -alias development -keyalg RSA -keysize 2048 -validity 10000
```

This keystore should be used for all the build variants except the one uploaded to the Play Store.

It should be committed to the repository together with the credentials in `build.gradle`, as explained in the official documentation.

## Production keystore

The production keystore **must** be protected with strong, random passwords. The keystore file **must** be committed to the repository, but the credentials will be saved in a password manager and downloaded as needed. The credentials **must not** be committed to the repository.

To create a production keystore, you can use Android Studio's built-in tool. Go to "Build" -> "Generate Signed Bundle / APK" -> "Next" -> "Create new...". Save the keystore somewhere in the project tree, as we want the keystore to be a part of the repository. The name for the keystore should be "package.name.of.your.project".
Then we need to choose a password. Try to use some random password generator (e.g. 1password) for this step and make sure to keep the password safe. This password is used both for the keystore and the signing/upload key itself, if you try to use different passwords you will get an error. Then we need to choose key alias, which can simply be the name of the project. The last thing you need to fill out is at least one of the fields under "Certificate" section. You can put "Infinum" at "Organization" for example.

![Creation of a new keystore file](/img/create_keystore_example.png)

After a successful creation of a keystore file and password, send the password and key alias to TL so it gets saved to a password manager. Commit the keystore file to the repository and push it to remote. If Vault is already setup for your project, then add password and key alias to it as well.

## App signing

App signing by Google Play is an opt-in feature and a recommended way to sign your app for distribution through Google Play. By using this feature, we add an extra layer of security to our application's release process. Also, this step is mandatory if we want to use the [Android App Bundle](https://developer.android.com/guide/app-bundle) and support [dynamic delivery](https://developer.android.com/guide/playcore/feature-delivery).

Now, we have to separate the **Upload** and **App Signing** key. The upload key is the one which developers use to sign and upload APK to Google Play. Google Play verifies the app and signs it with a new key that is used in production. Google Play is now responsible for storing the main app signing keystore. By separating the upload and the app signing key, you have the possibility to reset an upload key in case it gets lost or compromised. In case you are still using a single key for production, if you lose the key, you also lose the ability to update your app.

![Signing an app with App Signing by Google Play](/img/app_signing.png)

You should follow the steps in the [official documentation](https://developer.android.com/studio/publish/app-signing#enroll) to enable app signing. When switching an existing app to app signing, you need to export the current key as `*.pepk` file and upload it to Google Play. In either case, make sure that we are using different upload and app signing certificates when you are switching the app or signing a new app. If you are asked to accept the terms before setting up the app signing, please contact the account owner (in Infinum's case, this is Tomislav Car) or ask your team leader or project manager for help.

One great thing when Google is managing your signing key is that in case of lost or leaked upload key, we can request a reset. Note that the reset action takes some time. You will have a 48-hour period before the key is reset, during which you will not be able to release a new version with an old upload key.
