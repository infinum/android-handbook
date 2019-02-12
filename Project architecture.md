While defining a software arhicteture we always want to achive certain characteristics. For our team the focus is on flexibility, reusability and testability. We are guided by these characteristics but are not bound to them. The main purpose of project arhcitecture is consitency among the project which ease up the knowleadge share and transition of the people between projects. We also prefer architecture that helps with standard framework issues or makes them easier to solve. This in fact was the huge motivation to ditch the old MVP and switch to a new MVVM project architecture.



## What is MVVM?

// TODO descirbe the basics of MVVM architecture and how it diff from MVP



## Architecture components

[Android architecture components](https://developer.android.com/topic/libraries/architecture/) are a collection of libraries that help you design robust, testable, and maintainable apps. They serve as a main guideline in composing our project architecture backbone. By using Android architecture componets we don't have to worry much about managing app's lifecycle or loading data into our UI. This helps us focus on important and cool stuff in the project.

### ViewModel

[ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel) is an important part of the architecture and it representes the pillar of our bussines logic. ViewModel main purpose is to store and mange data for UI. It is designed to survive the activity configuration changes from which the screen orientation is the most common and troubles one. The below image displays scope of the ViewModel and it lifetime in memory with lifecycle states of an activity as it undergoes a rotation and then is finished.

![MVVM lifecycle](/img/mvvm_lifecyle.png "MVVM lifecycle")

 ViewModel might sound powerful but infect in the architecture components library this is just a simple abstract class.The magic stuff resides in the [ViewModelProvider](https://developer.android.com/reference/android/arch/lifecycle/ViewModelProvider). ViewModelProvider is responsible for ViewModel creation and retention or the ViewModel. To create ViewModelProvider for specific Activity/Fragment you should use  [ViewModelProviders](https://developer.android.com/reference/android/arch/lifecycle/ViewModelProviders) util class.

### LiveData

[LiveData](https://developer.android.com/topic/libraries/architecture/livedata) is lifecycle-aware observable class that is used as medium for data transfer from ViewModel to the View. Because of its connection to the activity lifecycle we have reduce number of memory leaks, crashes, lifecycle synchronization events beetween architecture layers. LiveData will take care of notifing the View about the latest data when the time is right. We only take care of preparing the correct data.

### SingleLiveData

// TODO



## What we solve with MVVM?

- **Canceling network calls**

Since we have LiveDate taking care of lifecycle state changes we don't have to worry that much about async work that loads our data. We can set the data to LiveData at any time without fear this is going to crash our app because the view is not ready. This means there is no need to cancel a network call when activity is moved to background. Thus no unecessary data traffic, repated calls, checks if we already have the data, inmemory cache in activity scope. 

- **Activity recreate and state preservation**

ViewModel together with all the data will survive the activity cnofiguration change. LiveData will again handle lifecycle state. View will be upto date. API will not be bother with any of this. Developer will be happy for the rest of its life. Ok lets not go so far we still need to use fragments :).

- **Flow control**

Most aplication contain some screens that can be grouped into a flow. Inside this flow you will often have a need to share some data. Information from the previous screen might be need or might influence some of the future screens. ViewModel and LiveDate can be a perfect solution for this type of situations. If you organize the specifc flow inside one activity and all other screens as fragment it is easy to share a ViewModel amongst all fragment and the controlling activity. This makes easy to share, store, combine and mange all the data in the flow. 



## The implementation

A school example of our proposal of MVVM architecture can be found on this [github repo](https://github.com/infinum/Android-MvvM-Example). In the Readme you will find all the details for base setup and starting to use Android archutecture componets.

// TODO other project repos using MVVM



