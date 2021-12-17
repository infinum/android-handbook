An Android App Bundle is a new upload format with the *.aab* file extension. A bundle will include all your app’s compiled code and resources, so you can upload it to Google Play just as a regular APK file. Google Play generates and signs an optimized APK for each user’s device configuration out of the uploaded bundle.

## Prepare

Before you start building a bundle make sure that you can and/or are allowed to *enable app signing by Google Play*.  
Check how to opt-in [here](https://support.google.com/googleplay/android-developer/answer/7384423?hl=en-GB).  
Otherwise, you can't upload your app bundle to the Play Console.

Configure bundle in your Gradle build script by enabling or disabling language, density and architecture split parameters.  
It is recommended to disable language split when your support language switching while application is running.  
If enabled, split will remove all but default and current device language from the bundle and break runtime language switch.  

```
android {

	...

    bundle {
        language {
            enableSplit = false
        }
        density {
            enableSplit = true
        }
        abi {
            enableSplit = true
        }
    }
}
```

Preferably, edit module configurations to create a bundle when running an application instead a default APK.  
![Edit configurations](/img/bundle_edit_configurations.png "Edit configurations") 

## Build

Easiest way to create a bundle is to use Android Studio 3.2 or higher.  
Bundle can be built via UI, from Build menu, or CLI, just like _assembleSomeFlavor_ tasks use a _bundleSomeFlavor_ task.

## Upload

Upload your app bundle to the Play Console just like a normal APK as before.

## Good to know

* Only devices running Android 5.0 and higher support downloading and installing dynamically generated APK files on demand.  
* These split APKs are very similar to normal APKs. They include compiled bytecode, resources, and an Android manifest. Android treats these  multiple installed split APKs as a single app.  
* Devices running Android 4.4 and lower don’t support downloading and installing split APKs. 
* They will be served a single APK, called a multi-APK which is *optimized for the device's configuration*. 
* Full app experience of using an app is preserved but APK does not include unnecessary code and resources incompatible with device requesting it, like other screen densities and CPU architectures. 
* You usually need to provide your SHA1 fingerprint in order to register at your API providers and obtain a key. Note that when using aabs, google will create a signing key for you and you will need to add its SHA1 to your API providers, else your API will stop working when going into production. The SHA1 from the key that google creates for you can be found on the Play console (Setup -> App integrity).
