Android requires that all APKs be digitally signed with a certificate before they can be installed.
You can find more details in the [official docs](https://developer.android.com/studio/publish/app-signing.html).

Once an app signed with a certificate is uploaded to the Play Store, **all updates to that app must be signed with the same certificate**.

The signing certificate is stored in a binary file known as the **keystore**.
The keystore is protected with a password and can store multiple certificates, each one protected with another password.

So to upload an update to the app you need:

- access to the Play Store publishing account
- the keystore which contains the certificate
- the keystore password and the certificate password

If a malicious user obtains all of these, he can upload whatever code he wants as an app update to unsuspecting users so it's very important to restrict access to this information.

It's also very important not to lose the keystore or passwords because then we lose the ability to update the app.

To do that, we use two separate keystores for each project:

- one for development and staging builds
- one for the production builds that will be uploaded to Play Store

## Development keystore

The development keystore should be generated using random alphanumeric passwords. You can do it via the Android Studio GUI or using [keytool](http://docs.oracle.com/javase/6/docs/technotes/tools/solaris/keytool.html):

```shell
keytool -genkey -v -keystore development.jks -alias development -keyalg RSA -keysize 2048 -validity 10000
```

This keystore should be used for all the build variants except the one which is uploaded to Play Store.

It should be commited to the repository along with the credentials in `build.gradle` like the official docs show.

## Production keystore

The production keystore **must** be protected with strong, random passwords. The keystore file **must** be commited to the repository, but the credentials will be saved in a password manager and downloaded as needed. The credentials **must not** be commited to the repository.

To make managing the production keystores easier we use an internal tool called [bombadil](https://bitbucket.org/infinum_hr/gem-bombadil).
Consult the docs on the link to see how to generate production keystores and use them.
