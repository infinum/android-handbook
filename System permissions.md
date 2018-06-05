Some parts of android framework have restricted access in order to protect critical data and code that could be misused. In order for your app to use such features you must declare permission in [AndroidManifest](http://developer.android.com/guide/topics/manifest/manifest-intro.html). Permissions are declared using `<uses-permission>` tag inside root `<manifest>` tag. Additionally if your app implements feature that is available for external use by other apps you can protect it with your own permission using `<permission>`tag.

Permissions example:

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

All permissions provided by android framework can be found at [Manifest.permission](http://developer.android.com/reference/android/Manifest.permission.html). They are separated into two lists. First one is list of `normal` permissions. Normal permissions don't pose much risk to the user's privacy or the device's operation. These permissions are automatically granted by system itself. You can find full list of normal permissions [here](http://developer.android.com/guide/topics/security/normal-permissions.html). All other permissions are listed as `dangerous` permission which means that they could potentially affect the user's privacy or the device's normal operation. You in the role of a developer must explicitly ask the user to grant these permissions.

In almost all cases, permission failure will be printed to the system log. In most cases this will also result with `SecurityException`.

## Permission groups

Every permission belongs to a permission group, including normal permissions and permissions defined by your app. Permission's group only affects the user experience if the permission is dangerous. You can ignore the permission group for normal permissions. When your app declares dangerous permissions system will warn user that app is trying to use permissions group that defined permission belongs to. Warning also contains short description what this permission group can do to your system. Below you can see table of dangerous permissions and permission groups.

![Permission group table](/img/permission_group_table.png "Permission group table")

## Permission request

Depending on android version system will ask for user permission to user restricted features differently.

For the device running Android 5.1 (API level 22) and lower this check will run at installation time. Before installing app system shows to user dialog with all permission groups this app needs access to. User needs to grant all permissions in order to continue with installation. User can revoke this permission only by uninstalling app.

![Permission pre M](/img/permission_check_old.png "Permission check on installation")

If the device is running Android 6.0 (API level 23) or higher app should request user's permission at run-time. This means that user can choose to allow or deny each permission group separately. User can also revoke permission at any time from app settings. This enables user to have full and flexible control over permissions app is using.

![Permission on M](/img/permission_check_new.png "Run-time permission check")

To enable run-time permission system developer needs to compile app with `targetSdkVersion` 23 or higher. If this is not the case system will still show permission dialog before installing the app. This doesn't stop user from revoking app permission in app settings but in this case system will notify user that denying permission may cause app to no longer function as intended.

![Compatibility warning](/img/permission_compatibilty_mode.png "Compatibility warning dialog on Android M")

## Handle run-time permissions

From Android 6.0 and higher, before the developer tries to access restricted part of android system he needs to check if user gave him permission to do that. In order to avoid writing something like this:

```java
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    ...
}
```

you should use `ActivityCompat` or `ContexCompat` support classes. This will ensure that method for checking permission grant status returns appropriate values for all system versions.

Before trying to use feature that is protected with dangerous permission `ALWAYS` check if user allowed usage of corresponding permission. To check this use `ContextCompat.checkSelfPermission(Context context, String permission)` method. As second parameter pass same permission label used in AndroidManifest for declaring permission. This method will return permission grant status that will match `PackageManager.PERMISSION_GRANTED` value if user allowed it. If return value doesn't match `PackageManager.PERMISSION_GRANTED` value you shouldn't proceed with this feature usage until user allows it. At that point you need to request permission from user. That is done with `requestPermissions(String[] permissions, int requestCode)` method. You need to pass String array containing all permission labels you want user to grant access. For every permission user will get request permission dialog that will enable user to allow or deny that permission.

![Permission request](/img/permission_request.png "Messenger run-time permission request")

You will receive user interaction result in `onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults)`. Parameter `grantResults` contains grant status for requested permissions. You need to compare its value with `PackageManager.PERMISSION_GRANTED` and if all values match `PackageManager.PERMISSION_GRANTED` you can finally access the needed feature. As you can see on the image above user can choose option "Never ask again". This means that user doesn't allow this permission for this app and he doesn't want you to request this permission again. If you try to request permission after user denied permission with never ask again option, system will not show request dialog to user but instead it will automatically send deny result back to you. In this case it is good to inform user why you need this permission and provide quick action to go on app settings detail screen where user can grant that permission.

We can now create base fragment that will do all the hard work for us. Then every fragment that needs to request permission can extend that base fragment. If you are requesting permission in `Activity` first consider moving logic in `Fragment`. If you have good reason why all the logic using restricted feature needs to be inside `Activity` to avoid [code duplication](https://en.wikipedia.org/wiki/Duplicate_code) you can use [headless fragment](http://luboganev.github.io/blog/headless-fragments/).

Example of base fragment for run-time permission handle:

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
     * @return True if user allowed all permissions, false otherwise
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
     * want to check and default request code is used.
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
     * Checks result of permission requests and calls permissionsGranted() if all permissions were allowed, otherwise it
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
     * implement custom logic. It also takes permissions and grantResults so that developer can handle
     * specific permission deny.
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
     * Checks if user allowed all the permissions that were requested.
     *
     * @param results Array of request results
     * @return True if user allowed all permissions, false otherwise
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
     * Displays default Snackbar with Action to open Application Details in Settings so the user
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
     * Creates intent for application details in settings.
     */
    protected final Intent getApplicationSettingsIntent() {
        Intent intent = new Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
        intent.setData(Uri.parse("package:" + getActivity().getPackageName()));
        return intent;
    }
}
```

In this example fragment that will extend `PermissionFragment` needs to implement `permissionGranted` method and can override `permissionDenied` to add custom behaviour when user denies requested permission. If the user wants to use custom logic, then he should override `requestPermissionResult` and use custom request codes. In extended fragment use `checkPermissions` method to preform permission grant status check. If method returns `false` call `requestPermission` method. To simplify checking and requesting permission you can use `checkAndRequestPermission` that will automatically request permissions if they are not granted.

`Pro tip:` If you have listed permission in app settings that you are not declaring in your AndroidManifest then one of library is declaring it. Sometimes it is not easy to detect which library added this ghost permission. To get some help you can look at `manifest-merger-report.txt` located in `build/outputs/logs` folder. If some lib added permission output should look something like this:

```plain
uses-permission#android.permission.GET_TASKS
ADDED from [co.infinum:NotificationLib:1.1.2]
```

For more information about run-time system permission see [Google I/O 2015 - Android M Permissions](https://www.youtube.com/watch?v=f17qe9vZ8RM) and [Android Marshmallow 6.0: Asking For Permission](https://www.youtube.com/watch?v=iZqDdvhTZj0)

#### Warning
- If you are using this Fragment as a child fragment inside a Viewpager, your fragment won't receive onRequestPermissionResult() callback. It happens because the Viewpager doesn't know to which fragment it has to send permission request result so you have to pass it yourself. One way to do it is to propagate callback inside Viewpager's host Fragment onRequestPermissionResult() using the code below.

```java  
for (Fragment f : getChildFragmentManager().getFragments()) {
    if (f instanceOf PermissionFragment) {
        ((PermissionFragment)f).onRequestPermissionResult(requestCode, permissions, grantResults);
    }
}
```

- You **MUST** include Android design support library in your project because PermissionFragment uses Snackbar to show messages
