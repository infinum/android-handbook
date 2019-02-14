While defining a software arhicteture we always want to achive certain characteristics. For our team the focus is on flexibility, reusability and testability. We are guided by these characteristics but are not bound to them. The main purpose of project arhcitecture is consitency among the project which ease up the knowleadge share and transition of the people between projects. We also prefer architecture that helps with standard framework issues or makes them easier to solve. This in fact was the huge motivation to ditch the old MVP and switch to a new MVVM project architecture.



## What is MVVM?

MVVM is a well known architecture that was publicly introduced in 2005 (source: [Introduction to Model/View/ViewModel pattern for building WPF apps](https://blogs.msdn.microsoft.com/johngossman/2005/10/08/introduction-to-modelviewviewmodel-pattern-for-building-wpf-apps/)). It consists of three concepts Model, View and ViewModel. On first glance it is very similar to MVP. 

* The Model in **M**VVM has the same role as it has in the MVP architecture. It is responsible for managing the data received from a specific datasource (Database, Network) completely UI independent.
* View in M**V**VM has also a similar role as in MVP - presenting the data to the user. In Android, View is usualy represented as an Activity, Fragment or an Android View. The core difference between the View in MVP and in MVVM is that the MVVM view is more knowladgable about the underlying Model and ViewModel in a sense that it knows about the events and states that have to be observed.
* ViewModel in MV**VM** is also known as the "Model of a View" and can be considered as a Views abstraction that creates a link between the Model and the View.

## Architecture components

[Android architecture components](https://developer.android.com/topic/libraries/architecture/) are a collection of libraries that help you design robust, testable, and maintainable apps. They serve as a main guideline in composing our project architecture backbone. By using Android architecture componets we don't have to worry much about managing app's lifecycle or loading data into our UI. This helps us focus on important and cool stuff in the project.

### ViewModel

[ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel) is an important part of the architecture and it representes the pillar of our bussines logic. ViewModel main purpose is to store and mange data for UI. It is designed to survive the activity configuration changes from which the screen orientation is the most common and troubles one. The below image displays scope of the ViewModel and it lifetime in memory with lifecycle states of an activity as it undergoes a rotation and then is finished.

![MVVM lifecycle](/img/mvvm_lifecyle.png "MVVM lifecycle")

ViewModel might sound powerful but in the architecture components library this is just a simple abstract class. The magic stuff resides in the [ViewModelProvider](https://developer.android.com/reference/android/arch/lifecycle/ViewModelProvider). ViewModelProvider is responsible for ViewModel creation and retention or the ViewModel. To create a ViewModelProvider for specific Activity/Fragment you should use [ViewModelProviders](https://developer.android.com/reference/android/arch/lifecycle/ViewModelProviders) util class.

### LiveData

[LiveData](https://developer.android.com/topic/libraries/architecture/livedata) is a lifecycle-aware observable class that is used as medium for data transfer from ViewModel to the View. Because of its connection to the activity lifecycle we have reduce number of memory leaks, crashes, lifecycle synchronization events beetween architecture layers. LiveData will take care of notifing the View about the latest data when the time is right. We only take care of preparing the correct data.

### SingleLiveEvent

Due to some use cases where LiveData was not working as expected (Snackbar, Navigation and other one shot events) Google added a custom implementation of LiveData called [SingleLiveEvent](https://github.com/googlesamples/android-architecture/blob/dev-todo-mvvm-live/todoapp/app/src/main/java/com/example/android/architecture/blueprints/todoapp/SingleLiveEvent.java) in the the [Android Architecture Blueprints](https://github.com/googlesamples/android-architecture#android-architecture-blueprints). It is a lifecycle-aware observable that sends only new updates after subscription, used for events like navigation and Snackbar messages. For more information regarding this topic please check this [article](https://medium.com/androiddevelopers/livedata-with-snackbar-navigation-and-other-events-the-singleliveevent-case-ac2622673150).



## What we solve with MVVM?

- **Canceling network calls**

Since we have LiveDate taking care of lifecycle state changes we don't have to worry that much about async work that loads our data. We can set the data to LiveData at any time without fear this is going to crash our app because the view is not ready. This means there is no need to cancel a network call when activity is moved to background. Thus no unecessary data traffic, repated calls, checks if we already have the data, inmemory cache in activity scope. 

- **Activity recreate and state preservation**

ViewModel together with all the data will survive the activity cnofiguration change. LiveData will again handle lifecycle state. View will be upto date. API will not be bother with any of this. Developer will be happy for the rest of its life. Ok lets not go so far we still need to use fragments :).

- **Flow control**

Most aplication contain some screens that can be grouped into a flow. Inside this flow you will often have a need to share some data. Information from the previous screen might be needed or might have influenced some of the future screens. ViewModel and LiveDate can be a perfect solution for these types of situations. If you organize the specifc flow inside one activity and all other screens as fragment it is easy to share a ViewModel between all fragment and the controlling activity. This makes it easy to share, store, combine and mange all the data in the flow. 


## The implementation

A school example of our proposal of MVVM architecture can be found on this [github repo](https://github.com/infinum/Android-MvvM-Example). In the Readme you will find all the details for base setup and starting to use Android archutecture componets.

// TODO other project repos using MVVM



