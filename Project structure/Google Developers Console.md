## Creating a project
Head over to [Developers console](https://console.developers.google.com), click the `Select a project` button on the top, right next to the page title and click `Create a project...`.

After you have created a new project in the Google Developers Console, you need to give access to your team leaders. To do so, open the `Permissions` page from the side menu, click `Add member` and add your team leaders' emails and select the `Is owner` permission.

## Enabling APIs

To use any Google APIs you might need in your app, such as Google Maps, Places, or Youtube API, you need to enable them in the Google Developers Console and generate an API key.

#### Google Maps for Android

1. Open the APIs page under APIs & auth in the side menu, select the API you want to enable and click `Enable API`.
2. Open the Credentials page under APIs & auth, click `Add credentials` -> `API key` - > `Android key`.
3. Enter the API key name and then add your application's package name and SHA-1 fingerprint.
   * Finding the SHA-1 fingerprint: In terminal, run the `keytool -list -v -keystore keystore_name.jks`command. The keystore should be located in the `app` folder of your project. `SHA1` is the fingerprint you need to copy and paste in the console. It will look something like this: `BB:0D:AC:74:D3:21:E1:43:67:71:9B:62:91:AF:A1:66:6E:44:5D:75`.
4. Click `Create` and the API key will be generated. It will look something like this: `AIzaSyBdVl-cTICSwYKrZ95SuvNw7dbMuDt1KG0`.
5. In `AndroidManifest.xml`, add the following element as a child of the `<application>` element, by inserting it just before the closing `</application>` tag:
```java
<meta-data
    android:name="com.google.android.geo.API_KEY"
    android:value="API_KEY"/>
```

And that's it! This is all you need to do in the Google Developers Console to be able to implement Google Maps in the app.
