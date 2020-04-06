There are two most common problems you will encounter when writing layout files:

1. You end up with a huge XML file
2. The same UI element is used on multiple screens

Large XML files are hard to debug, and copying the same UI element across the app is just stupid. This is why we use styles.

A style is a collection of properties that specify the look and format for a View or window. A style can specify properties, such as padding, font color, font size, background color, and much more. A style is defined in an XML resource called `styles.xml`.

## Example

Let’s say that you want to declare a button style that is used multiple times in the app. You declare its style in `styles.xml` like this:

```xml
  <style name="ConfirmationButton">
      <item name="android:textSize">@dimen/default_text_size</item>
      <item name="android:padding">@dimen/default_padding</item>
      <item name="android:layout_marginLeft">@dimen/default_margin</item>
      <item name="android:layout_marginRight">@dimen/default_margin</item>
      <item name="android:layout_marginTop">@dimen/default_margin</item>
      <item name="android:layout_gravity">center</item>
    </style>
```

After declaring the style, all you need to do is add it to a `Button` in the XML file.

```xml
 <Button android:text="@string/button_confirm_password_change"
         android:layout_width="match_parent"
         android:layout_height="wrap_content"
         style="@style/ConfirmationButton">
```

## Why, what, when?

Styles are not easy and, in some cases, they do not work as you thought they would. For detailed info on how and when to use them, read [this blog post](http://blog.danlew.net/2014/11/19/styles-on-android/).

## TextView vs Button

By definition, a [TextView](https://developer.android.com/reference/android/widget/TextView) is a user interface element that displays text to the user. On the other side, [Button](https://developer.android.com/reference/android/widget/Button) is a special TextView (extends TextView class) that presents user interface element the user can tap or click to perform an action. Developers usually misuse TextView element and make it act as Button. If your TextView is clickable, please use Button with proper style instead. If this practice is not followed, we don't know which TextViews in layouts are clickable and which are not, we need to open Java/Kotlin file where the layout is used to check if there are some click listeners set to it. Using this practice it is more straight-forward.

