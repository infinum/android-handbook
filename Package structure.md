At Infinum, we use package structure called `package by feature`.

## What is package by feature
It's an organizational style in which you group your code by functionalities rather than by layers. With package by feature, items that work closely together are placed next to one another, and that makes navigating through the code a lot easier.

## Example  

```
MyAwesomeApp (Application)
* di
* data
* shared
* ui
```

The `di` stands for *Dependency Injection*, and it's a package that contains the AppComponent and general Dagger modules that do not relate to specific screens (for example, ApiModule, ClientModule, AppContextModule, ExecutorsModule, etc.).

```
di
|-- component
|    |-- AppComponent
|-- module
     |-- ApiModule (Retrofit and ApiService)
     |-- ClientModule (provides OkHttpClient)
     |-- AppContextModule (provides Resources and Context)
     |-- ExecutorsModule (provides default executor)
     |-- HostModule (provides urls)

```

The `data` package contains all data management related code, which is grouped as interactors, models, and networking.

```
data
|-- interactors
|    |-- impl
|    |    |-- LoginInteractor
|    |    |-- RegisterInteractor
|    |-- LoginInteractor
|    |-- RegisterInteractor
|-- models
|    |-- body
|    |     |-- TransactionBody
|    |-- response
|    |     |-- TransactionResponse
|-- network
     |-- converters
     |-- ApiService
     |-- BaseCallback
     |-- RetrofitInteractor
     |-- RequestInterceptor
```

The `shared` package contains common utils and interfaces.

```
shared
|-- Constants
|-- interfaces
|    |-- AppComponent
|-- utils
     |-- ActivityUtil
```

The `ui` package contains subpackages named after the screen name (each screen is a separate feature). In general, the presenter and view code is contained in the `ui` subpackages. For each feature, besides the activity or fragment, the package contains presenter implementation, the MVP interface for the presenter and view (it is more `vp` than `mvp`, but we still stick to the `Mvp` prefix as it has recognizable meaning), and a `di` subpackage which contains a Dagger component and module for that screen. Features can also have subfeatures. For example, the `userinfo` feature consists of two subfeatures—`typeA` and `typeB`.

```
ui
|-- welcome
|-- login
|    |-- di
|    |    |-- LoginComponent
|    |    |-- LoginModule
|    |-- LoginActivity
|    |-- LoginPresenter (this is the implementation)
|    |-- MvpLogin
|-- userinfo
|    |-- typeA
|    |    |-- di
|    |    |    |-- ...
|    |-- typeB
|    |    |-- di
|    |    |    |-- ...
|    |-- ...
|    |  
|-- ...
...
```
