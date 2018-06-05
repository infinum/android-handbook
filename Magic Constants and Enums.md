## Consider using Magic Constants over Enums

More often than not you'll need to define a definite dataset describing states,
types or actions for closely related data type. Usually, Java developers would
do this using enumerated types known as **Enums**.
However, Android environment is much more strict regarding performance, and Enums consume more resources. Consider using **Magic Constants** instead.

### What are Magic Constants?

Magic constants are regular Java _public static final_ `ints` and `Strings`
specifically marked as enumerated values, then used as concrete return values
or method parameters. These constants are not written differently than any
other Java constant. Instead, values are enumerated in custom created
annotation.

### How to use Magic Constants?

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

In above example, public **setter** for field `activationStatus` of class `User`
receives an _int_ value as a method parameter. Class also defines three
conveniently named constants that this method obviously should receive.

However, nothing is stopping us to set **virtually any** integer.

```java
User user = new User();
user.setActivationStatus(25);
```

Example above is technically correct and compiler will not complain about it.
It is, however, not correct since unknown value is provided as parameter.
Magic Constants help with this issue, giving Android Studio and other tools
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

![Magic Constant not recognized error](/img/magic_constants_error_1.png "Magic Constant not recognized error")

As of Android Gradle Plugin version 1.3, checks are also available with the
command line lint tool, which brings this feature to CI.

![Magic Constants Lint CLI tool error report](/img/magic_constants_error_report_2.png "Magic Constants Lint CLI tool error report")

### Why - impact of Magic Constants compared to Enums

_Magic Constants_ is a fancy name for simple `integer` and `String` constants.
Enums are classes which need to be loaded at the time they are called. Each
enumerated value in Enum class needs to be instantiated. Each instance takes
memory for their ordinal and name, and reference on top of that. In the most
simplest form, Enum instance can take between **40** and **70 bytes**. This
number is multiplied by the number of instances.

Comparatively, `int` takes **4 bytes** of memory. Multiplying that number with
total number of constants equals to total memory usage.

In above example, _User_ class has three `int` constants, which total to 12
bytes of memory. Turning these constants to Enum values could take from
120 to over 200 bytes of memory.
