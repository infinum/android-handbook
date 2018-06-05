MVP stands short for Model-View-Presenter and it is a derivative from the MVC (Model View Controller) design pattern. The main difference between the two is shown in the picture below.

![MVC vs. MVP](/img/mvc_vs_mvp.png "MVC vs. MVP")


## What is MVP?

MVP pattern is a way to separate background tasks from Activities/Fragments/Views to make the application simpler, code shorter and maintainability better. MVP does all this by splitting the logic into 3 layers:

* **View** layer displays data to the user and reacts to user actions
* **Model** layer is a data access layer such as database API
* **Presenter** layer contains application logic and provides View with data from the Model


## Why use MVP?

The main reason to use MVP pattern is [KISS principle](https://people.apache.org/~fhanik/kiss.html). Without using MVP, your Activity contains UI, UI logic and data management which means lots of lines of code that are hard to maintain and cannot be reused. MVP splits complex tasks into multiple simpler tasks so they becomes easier to solve, classes have less code which means they become much easier to maintain, debug and understand for your colleagues. MVP also allows unit testing because you can individually test each component of the app by recreating real use cases with mock data.

## Model

Model is representation of business logic or simplified: data that will be displayed in the user interface (view). It is implemented by interactors that are responsible for some specific action (fetching data from API, reading from a file...) after which the result is sent to the presenter via listener.

## View

View is an interface that displays data (model) and is responsible for handling user input and invoking the corresponding method of the presenter. The view does only what the presenter tells the view to do and it shouldn't contain any logic in it.

## Presenter

The presenter is the “middle-man” and has references to view and model. For instance, if your app needs to display data of a user, the presenter would use its reference to a database business logic from where it would query a list of Users and find the one it needs. Then, it will call a method in View, passing the queried user and the View would update the UI with the user data.

## Listener

Listener is an interface implemented inside presenter and is used for retrieving data from the model. As model is usually asynchronous, presenter needs a way to listen when data is successfully fetched or if some kind of error happened. Listener is passed to the interactor as a method parameter and it is not injected via Dagger like other components.


## How to implement MVP?

* Create `mvp` package and `views`, `presenters`, `interactors` and `listeners` packages inside of it. Then add an **interface** inside every package that represents its actions. The picture below shows the proper way to organize your packages.

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

* Create `impl` package inside of `interactors` and `presenters` and create presenter and interactor class that implement their belonging interface and implement view interface in your Activity. Full code example is below.

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

* To successfully finish the implementation you have to pass all the references that are needed for successful communication. That is done by using [Dagger2](http://google.github.io/dagger/) for [dependency injection](http://stackoverflow.com/questions/130794/what-is-dependency-injection). You can find more about it [here](/android/dagger).


## Useful links

* [MVP for Android](http://antonioleiva.com/mvp-android/)
* [Introduction to MVP on Android](https://github.com/konmik/konmik.github.io/wiki/Introduction-to-Model-View-Presenter-on-Androids)
* [MVP Project Example](https://github.com/antoniolg/androidmvp)
