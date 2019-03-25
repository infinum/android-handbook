## Consider using magic constants over enums

More often than not, you'll have to define a definite dataset describing states,
types, or actions for a closely related data type. Java developers would usually
do this using enumerated types known as **enums**.
However, the Android environment is much stricter regarding performance, and enums consume more resources. Consider using **magic constants** instead.

### What are magic constants?

Magic constants are regular Java _public static final_ `ints` and `Strings`
specifically marked as enumerated values, then used as concrete return values
or method parameters. These constants are not written differently from any
other Java constant. Instead, values are enumerated in a custom
annotation.

### How to use magic constants?

```java
public class User {
  public static final int ACTIVATION_STATUS_ACTIVATED = 0;
  public static final int ACTIVATION_STATUS_DEACTIVATED = -1;
  public static final int ACTIVATION_STATUS_PENDING_ACTIVATION = -2;

  private String userName;
  private String password;
  private int activationStatus;

  public void setActivationStatus(int activationStatus) {
    this.activationStatus = activationStatus;
  }
}
```

In the above example, the public **setter** for the `activationStatus` field of the `User`
class receives an _int_ value as a method parameter. The class also defines three
conveniently named constants that this method should obviously receive.

However, nothing is stopping us from setting **virtually any** integer.

```java
User user = new User();
user.setActivationStatus(25);
```

The above example is technically correct, and the compiler will not complain about it.
However, it is not correct since an unknown value is provided as a parameter.
Magic constants help with this issue, giving Android Studio and other tools
enough information about specific values.

```java
public class User {
  public static final int ACTIVATION_STATUS_ACTIVATED = 0;
  public static final int ACTIVATION_STATUS_DEACTIVATED = -1;
  public static final int ACTIVATION_STATUS_PENDING_ACTIVATION = -2;

  @Retention(RetentionPolicy.CLASS)
  @IntDef({
    ACTIVATION_STATUS_ACTIVATED,
    ACTIVATION_STATUS_DEACTIVATED,
    ACTIVATION_STATUS_PENDING_ACTIVATION
  })
  public @interface UserActivationStatus { }

  private String userName;
  private String password;
  private int activationStatus;

  public void setActivationStatus(@UserActivationStatus int activationStatus) {
    this.activationStatus = activationStatus;
  }
}
```

Calling the same setter now gives an error in Android Studio.

![Magic constant not recognized error](/img/magic_constants_error_1.png "Magic Constant not recognized error")

As of Android Gradle Plugin version 1.3, checks are also available with the
command line lint tool, which brings this feature to CI.

![Magic Constants Lint CLI tool error report](/img/magic_constants_error_report_2.png "Magic Constants Lint CLI tool error report")

### Why—the impact of magic constants compared to enums

_Magic constants_ is a fancy name for simple `integer` and `String` constants.
Enums are classes which need to be loaded at the time they are called. Each
enumerated value in an enum class has to be instantiated. Each instance takes up
memory for their ordinal and name, and reference on top of that. In its 
simplest form, an enum instance can take up between **40** and **70 bytes**. This
number is multiplied by the number of instances.

Comparatively speaking, `int` takes up **4 bytes** of memory. Multiplying that number with
the total number of constants equals the total memory usage.

In the above example, the _User_ class has three `int` constants, which total to 12
bytes of memory. Turning these constants into enum values could take up from
120 to over 200 bytes of memory.
