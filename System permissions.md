Some parts of the Android framework have restricted access in order to protect critical data and code that could be misused. In order for your app to use such features, you have to declare a permission in [AndroidManifest](http://developer.android.com/guide/topics/manifest/manifest-intro.html). Permissions are declared using the `<uses-permission>` tag inside the root `<manifest>` tag. Additionally, if your app implements a feature that is available for external use by other apps, you can protect it with your own permission using the `<permission>`tag.

A permission example:

```xml
<manifest . . . >
    <permission android:name="com.example.project.DEBIT_ACCT" . . . />
    <uses-permission android:name="com.example.project.DEBIT_ACCT" />
    <uses-permission android:name="android.permission.INTERNET"/>
    <uses-permission android:name="android.permission.CAMERA"/>
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
    . . .
    <application . . .>
        <activity android:name="com.example.project.FreneticActivity"
                  android:permission="com.example.project.DEBIT_ACCT"
                  . . . >
            . . .
        </activity>
    </application>
</manifest>
```

## Permission levels

All permissions provided by the Android framework can be found at [Manifest.permission](http://developer.android.com/reference/android/Manifest.permission.html). They are separated into two lists. The first one is a list of `normal` permissions. Normal permissions don't pose a serious risk to the user's privacy or the device's operation. These permissions are automatically granted by the system itself. You can find the full list of normal permissions [here](http://developer.android.com/guide/topics/security/normal-permissions.html). All other permissions are listed as `dangerous` permissions, which means that they could potentially affect the user's privacy or the device's normal operation. You, as the developer, must explicitly ask the user to grant these permissions.

In almost all cases, a permission failure will be printed to the system log. In most cases, this will also result in `SecurityException`.

## Permission groups

Every permission belongs to a permission group, including normal permissions and permissions defined by your app. A permission's group only affects the user experience if the permission is dangerous. You can ignore the permission group for normal permissions. When your app declares dangerous permissions, the system will warn the user that the app is trying to use a permission group that the defined permission belongs to. The warning also contains a short description of what this permission group can do to your system. You can see a table of dangerous permissions and permission groups below.

![Permission group table](/img/permission_group_table.png "Permission group table")

## Permission request

Depending on the Android version, the system will ask for the user's permission to user-restricted features differently.

For a device running Android 5.1 (API level 22) and lower, this check will run at installation time. Before installing the app, the system shows a dialog with all permission groups this app needs access to. The user has to grant all permissions in order to continue with installation. The user can revoke this permission only by uninstalling the app.

![Permission pre M](/img/permission_check_old.png "Permission check on installation")

If the device is running Android 6.0 (API level 23) or higher, the app should request the user's permission at runtime. This means that the user can choose to allow or deny each permission group separately. The user can also revoke a permission at any time from the app settings. This enables the user to have full and flexible control over permissions the app is using.

![Permission on M](/img/permission_check_new.png "Run-time permission check")

To enable the runtime permission system, the developer has to compile the app with `targetSdkVersion` 23 or higher. If this is not the case, the system will still show a permission dialog before installing the app. This doesn't stop the user from revoking the permission in the app settings, but in this case, the system will notify the user that denying a permission may cause the app to no longer function as intended.

![Compatibility warning](/img/permission_compatibilty_mode.png "Compatibility warning dialog on Android M")

## Handle runtime permissions

From Android 6.0 and higher, before the developer tries to access a restricted part of the Android system, they have to check if the user gave them permission to do that. In order to avoid writing something like this:

```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    ...
}
```

you should use the `ActivityCompat` or `ContexCompat` support classes. This will ensure that the method for checking the permission grant status returns appropriate values for all system versions.

Before trying to use a feature that is protected with a dangerous permission, `ALWAYS` check if the user allowed the use of a corresponding permission. Use the `ContextCompat.checkSelfPermission(Context context, String permission)` method to check this. As the second parameter, pass the same permission label used in AndroidManifest for declaring permissions. This method will return a permission grant status that will match the `PackageManager.PERMISSION_GRANTED` value if the user allowed it. If the return value doesn't match the `PackageManager.PERMISSION_GRANTED` value, you shouldn't proceed to using this feature until the user allows it. At that point, you have to request a permission from the user. You do this with the `requestPermissions(String[] permissions, int requestCode)` method. You need to pass a String array containing all permission labels you want the user to grant access to. For every permission, the user will get a request permission dialog that will enable the user to allow or deny that permission.

![Permission request](/img/permission_request.png "Messenger run-time permission request")

You will receive the user interaction result in `onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults)`. The `grantResults` parameter contains the grant status for the requested permissions. You have to compare its value with `PackageManager.PERMISSION_GRANTED`. If all values match `PackageManager.PERMISSION_GRANTED`, you can finally access the necessary feature. As you can see in the above image, the user can choose the "Never ask again" option. This means that the user doesn't allow this permission for this app, and they don't want you to request this permission again. If you try to request permission after the user denied permission with the "Never ask again" option, the system will not show the request dialog to the user, but it will automatically send the denial result back to you instead. In this case, it is a good idea to inform the user why you need this permission and provide quick action to go to the app settings detail screen, where the user can grant that permission.

We can now create a base fragment that will do all the hard work for us. Then, every fragment that needs to request permission can extend that base fragment. If you are requesting permission in `Activity` first, consider moving logic in `Fragment`. If you have a good reason why all logic using restricted features has to be inside `Activity`, you can use the [headless fragment](http://luboganev.github.io/blog/headless-fragments/) to avoid [code duplication](https://en.wikipedia.org/wiki/Duplicate_code).

Here's an example of a base fragment for handling runtime permissions:

```java
public abstract class PermissionFragment extends BaseFragment {

    /**
     * Default Permission Request Code.
     */
    protected static final int DEFAULT_PERMISSION_REQUEST = 142;


    /**
     * Takes an array of permissions and checks if the user allowed all of them.
     *
     * @param permissions Array of permissions to check
     * @return True if user allowed all permissions, false otherwise
     */
    protected final boolean checkPermissions(String... permissions) {
        if (getActivity() != null) {
            for (String permission : permissions) {
                if (ActivityCompat.checkSelfPermission(getActivity(), permission) != PackageManager.PERMISSION_GRANTED) {
                    return false;
                }
            }
            return true;
        }
        return false;
    }

    /**
     * Takes an array of permissions and checks if the user allowed all of them.
     * If not, it automatically requests them with default request code.
     *
     * @param permissions Array of permissions to check and request if needed.
     * @return True if user allowed all permissions, false otherwise
     */
    protected final boolean checkAndRequestPermissions(String... permissions) {
        return checkAndRequestPermissions(DEFAULT_PERMISSION_REQUEST, permissions);
    }

    /**
     * Takes an array of permissions and checks if the user allowed all of them.
     * If not, it automatically requests them with custom request code.
     *
     * @param requestCode Request code that is used if permission is requested. MUST OVERRIDE requestPermissionResult() method if
     *                    request code is different than DEFAULT_PERMISSION_REQUEST.
     * @param permissions Array of permissions to check and request if needed
     * @return True if the user allowed all permissions, false otherwise
     */
    protected final boolean checkAndRequestPermissions(int requestCode, String... permissions) {
        if (checkPermissions(permissions)) {
            return true;
        } else {
            requestPermissions(permissions, requestCode);
            return false;
        }
    }


    /**
     * Simplified requestPermissions where you don't have to create new String[]{} but simply pass all permissions that you
     * want to check, and default request code is used.
     *
     * @param permissions Array of permissions that you want to request
     */
    protected void requestPermissions(String... permissions) {
        requestPermissions(permissions, DEFAULT_PERMISSION_REQUEST);
    }


    /**
     * Simplified requestPermissions where you don't have to create new String[]{} but simply pass all permissions that you
     * want to check.
     *
     * @param requestCode Request code used in onRequestPermissionResult. MUST OVERRIDE requestPermissionResult() method if
     *                    request code is different than DEFAULT_PERMISSION_REQUEST.
     * @param permissions Array of permissions that you want to request
     */
    protected void requestPermissions(int requestCode, String... permissions) {
        requestPermissions(permissions, requestCode);
    }


    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        if (requestCode == DEFAULT_PERMISSION_REQUEST) {
            defaultPermissionsResult(permissions, grantResults);
        } else {
            requestPermissionsResult(requestCode, permissions, grantResults);
        }
    }

    /**
     * Checks the result of permission requests and calls permissionsGranted() if all permissions are allowed, otherwise it
     * calls permissionDenied().
     */
    private void defaultPermissionsResult(String[] permissions, int[] grantResults) {
        if (checkPermissionResults(grantResults)) {
            permissionGranted();
        } else {
            permissionDenied(permissions, grantResults);
        }
    }

    /**
     * Called when all permissions are allowed. Must be implemented in a child.
     */
    protected abstract void permissionGranted();


    /**
     * Called when one of the permission is not allowed and shows Snackbar by default. Override this method to
     * implement custom logic. It also takes permissions and grantResults so that the developer can handle
     * specific permission denial.
     *
     * @param permissions  Array of asked permissions
     * @param grantResults Array of user responses
     */
    protected void permissionDenied(String[] permissions, int[] grantResults) {
        showSnackbar();
    }

    /**
     * Must override this method if using custom request code.
     *
     * @param requestCode  Custom request code to split the logic inside the method
     * @param permissions  Array of asked permissions
     * @param grantResults Array of user responses
     */
    protected void requestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        defaultPermissionsResult(permissions, grantResults);
    }

    /**
     * Checks if the user allowed all permissions that were requested.
     *
     * @param results Array of request results
     * @return True if thenuser allowed all permissions, false otherwise
     */
    protected final boolean checkPermissionResults(int[] results) {
        for (int result : results) {
            if (result != PackageManager.PERMISSION_GRANTED) {
                return false;
            }
        }
        return true;
    }

    /**
     * Displays the default Snackbar with Action to open the Application Details in Settings so the user
     * can enable permissions.
     */
    protected void showSnackbar() {
        if (getActivity() != null) {
            Snackbar.make(getActivity().findViewById(android.R.id.content), R.string.no_permission, Snackbar.LENGTH_LONG)
                    .setAction(R.string.action_settings, new View.OnClickListener() {
                        @Override
                        public void onClick(View v) {
                            startActivity(getApplicationSettingsIntent());
                        }
                    }).show();
        }
    }

    /**
     * Creates an intent for the application details in settings.
     */
    protected final Intent getApplicationSettingsIntent() {
        Intent intent = new Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
        intent.setData(Uri.parse("package:" + getActivity().getPackageName()));
        return intent;
    }
}
```

In this example, the fragment that will extend `PermissionFragment` has to implement the `permissionGranted` method and can override `permissionDenied` to add custom behavior when the user denies the requested permission. If the user wants to use custom logic, they should override `requestPermissionResult` and use custom request codes. In the extended fragment, use the `checkPermissions` method to carry out the permission grant status check. If the method returns `false`, call the `requestPermission` method. To simplify checking and requesting permissions, you can use `checkAndRequestPermission` that will automatically request permissions if they are not granted.

`Pro tip:` If you have listed a permission in the app settings that you are not declaring in your AndroidManifest, one of the libraries is declaring it. Sometimes it is not easy to detect which library added this ghost permission. If you need some help, you can have a look at `manifest-merger-report.txt` located in the `build/outputs/logs` folder. If some lib added a permission, the output should look something like this:

```plain
uses-permission#android.permission.GET_TASKS
ADDED from [co.infinum:NotificationLib:1.1.2]
```

For more information about runtime system permissions, see [Google I/O 2015 - Android M Permissions](https://www.youtube.com/watch?v=f17qe9vZ8RM) and [Android Marshmallow 6.0: Asking For Permission](https://www.youtube.com/watch?v=iZqDdvhTZj0)

#### Warning
- If you are using this Fragment as a child fragment inside a ViewPager, your fragment won't receive an onRequestPermissionResult() callback. This happens because the ViewPager doesn't know to which fragment it has to send the permission request result, so you have to pass it yourself. One way to do this is to propagate a callback inside the ViewPager's host Fragment onRequestPermissionResult() using the code below.

```java  
for (Fragment f : getChildFragmentManager().getFragments()) {
    if (f instanceOf PermissionFragment) {
        ((PermissionFragment)f).onRequestPermissionResult(requestCode, permissions, grantResults);
    }
}
```

- You **MUST** include the Android Design Support Library in your project because PermissionFragment uses Snackbar to show messages.
