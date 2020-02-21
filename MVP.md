## ~~Deprecated~~

The Android team has moved past the MVP architecture and is now using MVVM on all new projects. We've kept this chapter as a reference since there is still a large number of projects that use the MVP structure. If you are starting a new project or refactoring the existing one, check the MVVM architecture setup described in the [Project architecture](/Project architecture.md) chapter.

MVP is short for Model-View-Presenter, and it is a derivative from the MVC (Model View Controller) design pattern. The main difference between the two is shown in the picture below.

![MVC vs MVP](/img/mvc_vs_mvp.png "MVC vs. MVP")


## What is MVP?

The MVP pattern is a way to separate background tasks from Activities/Fragments/Views to make the application simpler, code shorter, and maintainability better. MVP does all this by splitting the logic into three layers:

* **View** layer displays data to the user and reacts to user actions
* **Model** layer is a data access layer, such as the database API
* **Presenter** layer contains application logic and provides the view with data from the model


## Why use MVP?

The main reason to use the MVP pattern is the [KISS principle](https://people.apache.org/~fhanik/kiss.html). Without using MVP, your Activity contains UI, UI logic, and data management. This means that there are many lines of code that are hard to maintain and cannot be reused. MVP splits complex tasks into multiple simpler tasks so they become easier to solve. Also, classes have less code, which means that they become much easier for your colleagues to maintain, debug and understand. MVP also allows unit testing because you can individually test each component of the app by recreating real use cases with mock data.

## Model

The Model is a representation of business logic or, to put it simpler, data that will be displayed in the user interface (view). It is implemented by the interactors that are responsible for some specific action (fetching data from the API, reading from a file...). After that, the result is sent to the presenter via listener.

## View

The view is an interface that displays data (model) and is responsible for handling user input and invoking the corresponding method of the presenter. The view does only what the presenter tells it to do, and it shouldn't contain any logic.

## Presenter

The presenter is the “middle-man” and has references to the view and model. For instance, if your app needs to display the data of a user, the presenter would use its reference to a database business logic. From there, it would query a list of users and find the one it needs. Then, it would call a method in the view, passing the queried user, and the view would update the UI with the user data.

## Listener

The listener is an interface implemented inside the presenter and used for retrieving data from the model. As the model is usually asynchronous, the presenter needs a way to listen when data is successfully fetched or when some kind of error happens. The listener is passed to the interactor as a method parameter, and it is not injected by Dagger like other components.


## How to implement MVP?

* Create a `mvp` package and the `views`, `presenters`, `interactors`, and `listeners` package inside of it. Then, add an **interface** inside every package that represents its actions. The picture below shows the proper way to organize your packages.

![Initial packages](/img/mvp_initial.png "Initial package setup")

```java
public interface LoginView {

    void hideProgress();

    void showProgress();

    void showError(String message);
}
```

```java
public interface LoginPresenter {

    void onLoginClick(String username, String password);
}
```

```java
public interface LoginInteractor {

    void login(LoginListener listener, String username, String password);
}
```

```java
public interface LoginListener {

    void onLoginSuccess (String message);

    void onLoginFail (String message);
}
```

* Create an `impl` package inside of `interactors` and `presenters`. Create a presenter and interactor class that implement their belonging interface and implement the view interface in your Activity. You can find an example of the full code below.

![MVP final packages](/img/mvp_final.png "MVP packages final form")

```java
public class LoginActivity extends AppCompatActivity implements LoginView {

    Button loginButton;
    EditText usernameEditText;
    EditText passwordEditText;
    ProgressDialog progressDialog;

    @Inject
    LoginPresenter mPresenter;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_login);
        loginButton = (Button) findViewById(R.id.button);
        loginButton.setOnClickListener(clickListener);
        initializeView();
    }

    private void initializeView() {
        usernameEditText = (EditText) findViewById(R.id.username);
        passwordEditText = (EditText) findViewById(R.id.password);
        DaggerLoginComponent.builder()
                .loginModule(new LoginModule(this))
                .build()
                .inject(this);
    }

    private View.OnClickListener clickListener = new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            String username = usernameEditText.getText().toString();
            String password = passwordEditText.getText().toString();
            mPresenter.onLoginClick(username, password);
        }
    };

    @Override
    public void hideProgress() {
        progressDialog.dismiss();
    }

    @Override
    public void showProgress() {
        if (progressDialog == null) {
            progressDialog = ProgressDialog
                    .show(this, "Loading", "Please Wait", true, false);
        } else {
            progressDialog.show();
        }
    }

    @Override
    public void showError(String message) {
        Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
    }

    @Override
    public void navigateToSplash() {
        Intent intent = new Intent(this, SplashActivity.class);
        startActivity(intent);
        finish();
    }
}
```

```java
public class LoginPresenterImpl implements LoginPresenter {

    private LoginView loginView;
    private LoginInteractor loginInteractor;

    @Inject
    public LoginPresenterImpl(LoginView loginView, LoginInteractor loginInteractor) {
        this.loginView = loginView;
        this.loginInteractor = loginInteractor;
    }

    @Override
    public void onLoginClick(String username, String password) {
        loginView.showProgress();
        loginInteractor.login(listener, username, password);
    }

    LoginListener listener = new LoginListener() {
        @Override
        public void onLoginSuccess(String message) {
            loginView.hideProgress();
            loginView.navigateToSplash();
        }

        @Override
        public void onLoginFail(String message) {
            loginView.hideProgress();
            loginView.showError(message);
        }
    };
}
```

```java
public class LoginInteractorImpl implements LoginInteractor {

    LoginListener listener;

    @Inject
    public LoginInteractorImpl() {
    }

    @Override
    public void login(LoginListener listener, String username, String password) {
        this.listener = listener;
        startDummyLogin();
    }

    private void startDummyLogin() {
        new Handler().postDelayed(new Runnable() {
            @Override
            public void run() {
                listener.onLoginSuccess("Success");
            }
        }, 4000);
    }
}
```

* You have to pass all references that are needed for a successful communication to finish the implementation. This is done by using [Dagger2](http://google.github.io/dagger/) for [dependency injection](http://stackoverflow.com/questions/130794/what-is-dependency-injection). You can find more about it [here](/android/dagger).


## Useful links

* [MVP for Android](http://antonioleiva.com/mvp-android/)
* [Introduction to MVP on Android](https://github.com/konmik/konmik.github.io/wiki/Introduction-to-Model-View-Presenter-on-Androids)
* [MVP Project Example](https://github.com/antoniolg/androidmvp)
