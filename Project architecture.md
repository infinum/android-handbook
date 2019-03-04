While defining a software architecture we always want to achieve certain characteristics. For our team, the focus is on flexibility, reusability, and testability. We are guided by these characteristics but are not bound to them. The main purpose of project architecture is consistency among the project which eases up the knowledge share and transition of the people between projects. We also prefer architecture that helps with standard framework issues or makes them easier to solve. This in fact was the huge motivation to ditch the old MVP and switch to a new MVVM project architecture.



## What is MVVM?

Today MVVM is a well known architecture that was developed by Microsoft and was publicly introduced in 2005 (source: [Introduction to Model/View/ViewModel pattern for building WPF apps](https://blogs.msdn.microsoft.com/johngossman/2005/10/08/introduction-to-modelviewviewmodel-pattern-for-building-wpf-apps/)). It consists of three components Model, View and ViewModel. Let us take a more detailed look at the components: 

* The Model in **M**VVM has the same role as it has in the MVP architecture. It is responsible for managing the data received from a specific data source (Database, Network), completely UI independent.
* View in M**V**VM has also a similar role as in MVP - presenting the data to the user. In Android, View is usually represented as an Activity, Fragment or an Android View. The core difference between the View in MVP and in MVVM is that the MVVM view is more knowledgable about the underlying Model and ViewModel in a sense that it knows about the events and states that have to be observed.
* ViewModel in MV**VM** is also known as the "Model of a View" and can be considered as a Views abstraction that creates a link between the Model and the View. The difference between the Presenter from MVP and the ViewModel is that the Presenter has a reference to a view, whereas the ViewModel directly binds to properties on the view model to send and receive updates.


![MVVM Architecture diagram](/img/mvvm_architecture.png, "MVVM Architecture diagram")


## Architecture components

[Android architecture components](https://developer.android.com/topic/libraries/architecture/) are a collection of libraries that help you design robust, testable, and maintainable apps. They serve as a main guideline in composing our project architecture backbone. By using Android architecture components we don't have to worry much about managing app's lifecycle or loading data into our UI. This helps us focus on important and cool stuff in the project.

### ViewModel

[ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel) is an important part of the architecture and it represents the pillar of our business logic. ViewModel main purpose is to store and manage data for UI. It is designed to survive the activity configuration changes from which the screen orientation is the most common and troubles one. The below image displays scope of the ViewModel and its lifetime in memory with lifecycle states of an activity as it undergoes a rotation and then is finished.

![MVVM lifecycle](/img/mvvm_lifecyle.png "MVVM lifecycle")

ViewModel might sound powerful but in the architecture components library, this is just a simple abstract class. The magic stuff resides in the [ViewModelProvider](https://developer.android.com/reference/android/arch/lifecycle/ViewModelProvider). ViewModelProvider is responsible for ViewModel creation and retention or the ViewModel. To create a ViewModelProvider for specific Activity/Fragment you should use [ViewModelProviders](https://developer.android.com/reference/android/arch/lifecycle/ViewModelProviders) util class.

### LiveData

[LiveData](https://developer.android.com/topic/libraries/architecture/livedata) is a lifecycle-aware observable class that is used as a medium for data transfer from ViewModel to the View. Because of its connection to the activity lifecycle we have reduced number of memory leaks, crashes, lifecycle synchronization events between architecture layers. LiveData will take care of notifying the View about the latest data when the time is right. We only take care of preparing the correct data.

### LiveEvent

Due to some use cases where LiveData was not best suited to handle the problem of resubscriptions (Snackbar, Navigation and other one shot events) Google added a custom implementation of LiveData called [SingleLiveEvent](https://github.com/googlesamples/android-architecture/blob/dev-todo-mvvm-live/todoapp/app/src/main/java/com/example/android/architecture/blueprints/todoapp/SingleLiveEvent.java) in the [Android Architecture Blueprints](https://github.com/googlesamples/android-architecture#android-architecture-blueprints). It is a lifecycle-aware observable that sends only new updates after subscription, used for events like navigation and Snackbar messages. For more information regarding this topic please check this [article](https://medium.com/androiddevelopers/livedata-with-snackbar-navigation-and-other-events-the-singleliveevent-case-ac2622673150).
However, the SingleLiveEvent implementation from Google has one major setback and that is the fact that it works only with one observer at a time. This issue is addressed and explained in detail in this [article](https://proandroiddev.com/livedata-with-single-events-2395dea972a8). The owner of the article proposes an improved implementation, called LiveEvent, which covers the case of multiple observers. The source for the implementation: [LiveEvent](https://github.com/hadilq/LiveEvent/blob/master/lib/src/main/java/com/hadilq/liveevent/LiveEvent.kt)


## What we solve with MVVM?

- **Canceling network calls**

Since we have LiveDate taking care of lifecycle state changes we don't have to worry that much about async work that loads our data. We can set the data to LiveData at any time without fear this is going to crash our app because the view is not ready. This means there is no need to cancel a network call when activity is moved to the background. Thus no unnecessary data traffic, repeated calls, checks if we already have the data, in-memory cache in activity scope. 

- **Activity recreate and state preservation**

ViewModel together with all the data will survive the activity configuration change. LiveData will again handle the lifecycle state. A view will be up to date. API will not be bothered with any of this. Developers will be happy for the rest of its life. Ok let's not go so far we still need to use fragments :).

- **Flow control**

Most applications contain some screens that can be grouped into a flow. Inside this flow you will often have a need to share some data. Information from the previous screen might be needed or might have influenced some of the future screens. ViewModel and LiveDate can be a perfect solution for these types of situations. If you organize the specific flow inside one activity and all other screens as a fragment it is easy to share a ViewModel between all fragment and the controlling activity. This makes it easy to share, store, combine and manage all the data in the flow. 


## The implementation

This section will go through some main guidelines on how to implement MVVM in your Android project. We will cover an implementation of MVVM using the architectural components from Android that we described in an earlier section.

The user interface in android app is made with a collection of View and ViewGroup objects that are inside of an Activity or a Fragment container. Those two containers represent the View component in the MVVM architecture. You may wonder why an ordinary Android View could not be the View in MVVM? This is a totally valid case and can be implemented in many ways, but we will focus on Activities and Fragments because they both implement [LifecycleOwner](https://developer.android.com/reference/android/arch/lifecycle/LifecycleOwner) interface which gives us the opportunity to use LiveData and ViewModel. The use of LiveData and ViewModel gives us a great head start in developing our MVVM architecture. 

Let us take a look at the ViewModel first. It is probably a good idea to have a base implementation of ViewModel that will handle some shared logic and reduce boilerplate in our codebase. This will look something like this:

```
open class BaseViewModel: ViewModel()
```
Now we need a means of communication between the ViewModel and the View. But before we talk about that let's see what kind of information our View expects in the communication. In other words, what should we expose to the View from the ViewModel? 

In most cases, the View wants to know about the state of the screen so that it can render the changes accordingly. The state of a screen is an object that represents the statefull data that the user can see on the screen. To better understand what is part of the screen state one can look at it in the following way. A part of the state can be any data representation of a UI component that we want the View to render after it was recreated, where the end goal is to have the same screen like the one before the View got recreated. 

Unlike states, there are things that we do not want to render after a View recreation. For example, display of a self dismissable messages (Toast, Snackbar, etc.) or some kind of navigation triggers. For this, we need to expose another piece of information to the View and we call that events. An event is an one shot trigger that is emitted from the ViewModel and consumed by the View only once. 

Considering the concepts of State and Event we would adjust our BaseViewModel implementation to be aware of them. The end result would look like this:

```
open class BaseViewModel<State : Any, Event : Any> : ViewModel() 
```
We now have the state and events for a specific View, but we are still missing a way to expose these objects to the View. This is where LiveData and LiveEvent implementations come into play. 
Inside ViewModel we need LiveData objects of State and Events that the View could observe. The LiveData objects would look something like this:

```
open class BaseViewModel<State : Any, Event : Any> : ViewModel() {

    private val stateLiveData: MutableLiveData<State> = MutableLiveData()
    private val eventLiveData: LiveEvent<Event> = LiveEvent()
    
    fun viewStateData(): LiveData<State> = stateLiveData
    fun viewEventData(): LiveData<Event> = eventLiveData
}
```
As we can see in the above code, we expose our private stateLiveData and eventLiveData through functions that return immutable LiveData. This way we make sure that we do not allow changes to our live data objects directly, but at the same time we allow observing changes made on our live data objects.

To make it easier to use the LiveData objects we can make a convenience backing field and function that will help us manipulate the LiveData objects in our concrete ViewModel implementation.

```
open class BaseViewModel<State : Any, Event : Any> : ViewModel() {
   
   //...
 	
 	protected var viewState: State? = null
        get() = stateLiveData.value ?: field
        set(value) {
            val viewState = stateLiveData.value
            stateLiveData.value = value
        }
        
    protected fun emitEvent(event: Event) {
        eventLiveData.value = event
    }
}
```

After we have a BaseViewModel implementation, for example LoginViewModel, we need to provide it to the View. This can be achieved using a utility method from the ViewModelProviders class: 

```
ViewModelProviders.of(this).get(LoginViewModel::class.java)

```

In the above code, `this` represents the LifecycleOwner. In our case, as mentioned earlier, this is either an Activity or Fragment. The above code returns a ViewModel instance which we use to observe the LiveData objects.

```
loginViewModel.viewStateData().observe(this, { state -> 
	// Update the UI state 
})

loginViewModel.viewEventData().observe(this, { event -> 
	// Handle event
})

```

This is now enough to have a connection between View and ViewModel.

So far we only concentrated on View and ViewModel, but what about the Model in MVVM? The Model is responsible for managing the data received from a specific data source (Database, Network, etc.), completely UI independent. This means that the model should expose its data only to the ViewModel. The ViewModel can also request some data from the Model, so it is two-way communication.

At this point, the idea of MVVM and how to implement it should be much clearer and easier to understand. Take into account that the code you saw was very simplified to only show the conceptual idea of MVVM in Android. This is just one way to implement it, there are many more different kinds of implementation that also include [DataBinding](https://developer.android.com/topic/libraries/data-binding/) or are very [Rx](http://reactivex.io/) heavy.

A school example of our proposal of MVVM architecture can be found on this [github repo](https://github.com/infinum/Android-MvvM-Example). In the Readme you will find all the details for base setup and starting to use Android architecture componets.

For more projects with MVVM architecture check out the following list:

* [HAK-ispitivaci](https://github.com/infinum/Android-Hak-Ispitivaci)
* [Porsche-Group-Slovenia](https://github.com/infinum/Android-Porsche-Group-Slovenia)
* [XtraCash](https://github.com/infinum/Android-XtraCash)



