When writing layout files there are two most common problems you will encounter:

1. You end up with a huge xml file
2. The same UI element is used on multiple screens

Large xml files are hard to debug and copying the same UI element across the app is just stupid. This is why we use styles.

A style is a collection of properties that specify the look and format for a View or window. A style can specify properties such as padding, font color, font size, background color, and much more. A style is defined in an XML resource called `styles.xml`.

## Example

Let’s say you want to declare a button style which is used multiple times in the app. You declare its style in `styles.xml` like this:

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

After declaring the style all you need to do is add it to a `Button` in xml file.

```xml
 <Button android:text="@string/button_confirm_password_change"
         android:layout_width="match_parent"
         android:layout_height="wrap_content"
         style="@style/ConfirmationButton">
```

## Why, what, when?

Styles are not easy, and in some cases they do not work as you thought they would. For detailed info on how and when to use them read [this blog post](http://blog.danlew.net/2014/11/19/styles-on-android/).
