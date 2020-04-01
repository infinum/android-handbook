Good naming makes it easier to read and understand code.

Consider the following two versions of functionally equivalent code.

```java
List<String> list1 = new ArrayList<>();
for (x : list1) {
    if (x.size() <= 4) {
        list1.add(x);
    }
}
return list1;
```

```java
List<String> shortFileNames = new ArrayList<>();
for (name : filenames) {
    if (name.size() <= FILE_NAME_LENGTH_LIMIT) {
        shortFileNames.add(name);
    }
}
return shortFileNames;
```

Which one is more understandable?

To make your code understandable, you need to `reveal your intent` and `avoid disinformation`.

Methods which return boolean values should sound like questions.

```java
if (isFileTooLarge(image))
```

is more readable than

```java
if (checkFileSize(image))
```

## Naming conventions

* All code is written in English, including comments.
* Upper `CamelCase` is used for classes.
* Lower `camelCase` is used for methods, local variables, and instance variables.
* Abbreviations and acronyms should use camel case as normal words (e.g., `XmlHttpRequest`, `QrCodeReader`, `userId`).
* Constants should be `ALL_CAPS`.
* Methods are named after verbs, variables and classes after nouns.
* XML resource IDs have to use `snake_case`.
* XML resource IDs should use `camelCase` in a Kotlin project because of [Kotlin synthetic](https://kotlinlang.org/docs/tutorials/android-plugin.html).
* Activities and fragments are named with the appropriate suffix: `LoginActivity`, `RegisterFragment`.
* The corresponding layouts have to be named with the appropriate prefix: `activity_login.xml`, `fragment_register.xml`.
* All naming has to be simple, clear and mnemonic (short and meaningful).
* The [Infinum Code Style](https://github.com/infinum/android-handbook-private/blob/master/files/InfinumCodeStyle.xml) has to be imported into the Android Studio and used in formatting code. If you want to find out how to import it, check out the chapter about [Development environment setup](/books/android/onboarding/development-environment-setup).
* Drawable resources also have [naming conventions](http://petrnohejl.github.io/Android-Cheatsheet-For-Graphic-Designers/#naming-conventions).

## Examples of good naming

```java
public class WelcomeActivity extends BaseActivity implements WelcomeView {

    public static final int REQUEST_CODE_LOGIN = 147;

    public static final String EXTRA_ACTION = "EXTRA_ACTION";

    @InjectView(R.id.viewExchangeQuote)
    protected ExchangeQuoteView exchangeQuoteView;

    @InjectView(R.id.textSeeDetails)
    protected TextView seeDetailsText;

    @OnClick(R.id.textSeeDetails)
    protected void onSeeDetailsClicked() {
        presenter.onDetailsRequested();
    }

    @OnClick(R.id.loginButton)
    protected void onLoginButtonClicked() {
        presenter.onLoginButtonClicked();
    }

    @OnClick(R.id.registerButton)
    protected void onRegisterButtonClicked() {
        presenter.onRegisterButtonClicked();
    }

    @Override
    public void showCurrencies(List<Currency> currencies) {
        exchangeQuoteView.showCurrencies(currencies);
    }

    @Override
    public void showSellingAmount(String sellingAmount) {
        exchangeQuoteView.showSellingAmount(sellingAmount);
    }

    @Override
    public void showBuyingAmount(String buyingAmount) {
        exchangeQuoteView.showBuyingAmount(buyingAmount);
    }

    @Override
    public void navigateToLogin() {
        Intent intent = new Intent(this, LoginActivity.class);
        startActivityForResult(intent, REQUEST_CODE_LOGIN);
    }

    @Override
    public void navigateToRegister() {
        Intent intent = RegistrationActivity.buildIntent(this);
        startActivityForResult(intent, REQUEST_CODE_REGISTER);
    }
}
```

```xml
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
            xmlns:tools="http://schemas.android.com/tools"
            android:layout_width="match_parent"
            android:layout_height="match_parent">

    <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">

        <co.infinum.currencyfair.custom.ExchangeQuoteView
                android:id="@+id/viewExchangeQuote"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:visibility="invisible"
                tools:visibility="visible"/>

        <com.ivankocijan.magicviews.views.MagicTextView
                android:id="@+id/textSeeDetails"
                style="@style/TextButton"
                android:padding="8dp"
                android:textAllCaps="false"
                tools:text="@string/see_how_you_save"
                android:visibility="invisible"
                tools:visibility="visible"/>

        <Space
                android:layout_width="match_parent"
                android:layout_height="0dp"
                android:layout_weight="1"/>

        <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal">

            <com.ivankocijan.magicviews.views.MagicButton
                    android:id="@+id/registerButton"
                    style="@style/AppTheme.Button"
                    android:layout_width="0dp"
                    android:layout_weight="0.5"
                    android:layout_marginRight="8dp"
                    android:text="@string/register"/>

            <Space android:layout_width="16dp"
                   android:visibility="gone"
                   android:layout_height="1dp"/>

            <com.ivankocijan.magicviews.views.MagicButton
                    android:id="@+id/loginButton"
                    style="@style/AppTheme.Button"
                    android:layout_width="0dp"
                    android:layout_weight="0.5"
                    android:text="@string/login"/>

        </LinearLayout>
    </LinearLayout>
</ScrollView>
```
