At Infinum we use package structure called `Package by feature`

## What is Package by feature
It's an organisational style in which you group your code by functionalities rather than by layer. With package by feature items that work closely together are placed next to each other and that makes navigating through the code a lot easier.

## Example  

```
MyAwesomeApp (Application)
* di
* data
* shared
* ui
```

The `di` stands for *Dependency Injection* and it's a package that contains the AppComponent and general Dagger modules that do not relate to specific screens (for example ApiModule, ClientModule, AppContextModule, ExecutorsModule, etc).

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

The `data` package contains all data management related code which is grouped as interactors, models, and networking.

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

The `ui` package contains subpackages named after the screen name (each screen is a separate feature). In general, presenter and view code is contained in the `ui` subpackages. For each feature, besides the activity or fragment, the package contains presenter implementation, mvp interface for presenter and view (it is more `vp` than `mvp`, but we still stick to the `Mvp` prefix for it has a recognizable meaning), and `di` subpackage which contains dagger component and module for that screen. Features can also have subfeatures. For example `userinfo` feature consists of two subfeatures `typeA` and `typeB`.

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
