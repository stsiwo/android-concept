# Android & Kotlin Basic Guide

## Constraint

there are specific constraints for android app:

- launch the app at any entry point (multiple entry point and out-of-order)
- any component should not depend on each other (multiple entry point and out-of-order)
- shouldn't store any app data or state in your app components (cannot control when your component is destroyed because of memory pressure and user want to kill)

## Architecture

1. __separation of concerns__: don't put all logic (application, ui, infrastructure) into Activity or Fragment class.

   minimize your dependency on Activity/Fragment for better UX, manageable app and maintatenance experience.

2. __drive UI from model (persistent)__

- because of memory pressure (users don't lose data if the OS destroy the app)
- becauese of no network connection (the app continue to work)
- model: a class what is responsibile for managing the data

### Recommended App Architecture

1. __Activity/Fragment (Controller)__
2. __ViewModel (Application)__
   
   __LiveData__

3. __Repository (Infra)__
4. __Model (Infra)__

   __Room__

5. __Remote Data Source (Infra)__

   __Retrofit__

   each component depends only on the component one level below it. this createa a consistent and pleasant user expereience.

#### Fragment

   a partial UI and a subset of Activity.

#### Activity

   a whole screen and often contain multiple Fragments.

#### ViewModel

   provides the data for a specific UI component, such as a fragment or activity, and contains data-handling business logic to communicate with the model.

   doesn't know about UI Components (good polymorphism: dependency does not need to know who is my dependee and vise versa) so it __isn't affected by configuration changes__, such as recreating an activity when rotating the device. 

   __manage any ansyc calls__ instead of UI controllers (Activity & Fragment). if UI controllers manage this, it might waste of resource since it might call async when re-creation but data is already fetch previous creation.

   with lifecycle conscious

   onCreate at the 2nd times on an activity return the same ViewModel object.

   when an activity finish its work, it also call 'onCleared' method on its related ViewModel for cleanup.

   __never reference a view, Lifecycle, or any class that may hold a rerence to the activity context__. ?? __why?__

   scoped to the Lifecycle object passed to the ViewModelProvider.

   use share the data to multiple Fragments on the activity.

   __SavedState__ allows ViewModel to access the saved state and arguments of the associated Fragment or Activity.

##### Keep UI Sync with Persistent Storage

use Room and LiveData

- Room: informs your LiveData when the database chagnes

- LiveData: updates your UI with the revised data
  - lifecycle-aware, meaning it respects the lifecycle of other app components to ensures LiveData only updates app component observers that are in an active lifecycle state
  - you usually use async with coroutine


#### save UI state

- ViewModel: store and manage UI-reated data (in-memory)

1. quickest access
2. avoid refetching data from network or disk or configuration change
3. destroyed when user backs out of your UI controllers or 'finish()' function or system-initiated process death. => you need to use 'onSaveInstanceState()' to prevent this so that you don't lose user data.

- onSaveInstanceState: for small amount of data (disk)

1. serialization (might take time)

- SavedStateRegistry: allow components to hook into your UI controller's saved state to consume or contribute to it. 

1. used for ViewModel to provide UI state to your viewModel
2. can retrieve this by calling 'getSavedStateRegistry()' at UI Controllers

- local persistence: survive for as long as your application is installed on the user's device (unless teh user intentionaly delete it)

#### Repository (Architecture)

abstract any persistent storage and its knowledge from ViewModel.

tight coupling with ViewModel with data fetching logic cause undesirable data loss (fetched data) this is because if the lifecyle of the ViewModel ends, the fetched data is also deleted. So, better to delegate the data fetching logic to different layer, which is a Repository.

also, decide which storage to go (e.g., Model/Remote Data Resource) if data is stale or not

ViewModel ====> Repository (mediator) ====> data sources (persistent models, web service, and caches)

define more narrower interface (function): 'get(id)', 'find(id)', 'add(entity)'

##### cache (in-memory)

use cache once you fetch the data from api because:

- it wastes valuable network bandwidth
- it forces the user to wait for the new query to complete.



#### Model (Architecture)

used for caching and if data is stale, ask Remote Data Source to fetch updated data.

hold domain ID and the domain object.

- domain ID: this data is perseved even if the OS destroys the app.

- domain object: a data class hold details about the domain.

#### Room

persistence library.

an object-mapping library that provides local data persistence with minimal boilerplate code.

it has annotaiton like @Entity (oh i miss Hibernate...), @PrimaryKey, @Database, and so on.

provide main-safety automatically with coroutine so you don't need to handle coroutines on your own.

#####  __data acess object (DAO)__

an abstraction of data persistence. this is a lower-level concept, closer to teh storage systems.

works as a data mapping/access layer, hiding ugly queries.

mapping with a table of database.

a Repository can use a DAO but a DAO cannot use a Repository since DAO is lower level concept.

define basic method like 'save', 'update', 'load'

##### Flow with Room

allows you to get live udpates. This means that every time there's a change in the user table, a new User will be emitted. called _observable data source_.

#### Retrofit (library)

use to fetch data from a API. technically, it is an API adapter wrapped over okHTTP (API fetching library) to map HTTP API endpoints to Java/Kotlin interface.

ex)
```
public interface GitHubService {
  @GET("users/{user}/repos") // your endpoint
  Call<List<Repo>> listRepos(@Path("user") String user); // a method to request to the endpoint
}

// config
Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://api.github.com/")
    .build();

// create a service based on the interface
GitHubService service = retrofit.create(GitHubService.class);

// now you can request and grab the result of Java/Kotlin object
Call<List<Repo>> repos = service.listRepos("octocat");
```

provide main-safety automatically with coroutine so you don't need to handle coroutines on your own.

#### network status handling

read this: https://developer.android.com/jetpack/guide#addendum


### Best Practice

#### app's entry points—such as activities, services, and broadcast receivers should focus on only coordindating with other components and should be short-lived.

don't use those components for data source.

#### don't duplicate a code for a logic more than once

each component/class has its own responsibility and don't put wrong logic to a wrong component/class

#### don't expose interal logic to outside the module

only expose necessary function/interface outside the component/class. exposing the logic causes you to double your work when you need to update/change the logic

#### don't reinvent the wheel

use existing library or built-in function as much as possible. don't re-implement code which already exist. instead, focus on core features of your app.

#### prepare for offline

persist as much relevant and fresh data as possible.

#### single source of truth

your database is the source of truth


### View Binding

a feature that allows you to more easily write code that interacts with views.

view file generate its binding class which can be use in its corresponding activity/fragment


### Data Binding Library

a support libarary taht allows you to bind UI components in your layouts to data sources in your app using a declarative format rather than programmatically.

In many cases, view binding can provide the same benefits as data binding with simpler implementation and better performance. If you are using data binding primarily to replace findViewById() calls, consider using view binding instead. ??

#### ViewBinding vs DataBinging

1. With view binding, the layouts do not need a layout tag

2. You can't use viewbinding to bind layouts with data in xml (No binding expressions, no BindingAdapters nor two-way binding with viewbinding)

3. The main advantages of viewbinding are speed and efficiency. It has a shorter build time because it avoids the overhead and performance issues associated with databinding due to annotation processors affecting databinding's build time.


### Paging library

latest version: 3

pagination feature with:

1. in-memory caching
2. built-in request deduplication
3. recyclerView apdapters: automatically request dat as the user scrolls toward the end of the loaded data.
4. error handling with refresh and retry capabilities.


#### PagingSource

defines a source of data and how to retrieve data from that source. it can load from any single source, like network sources and local databases.

#### RemoteMediator

handles paging from a layered data source, such as a network data source with a local database cache.

#### Pager 

provide public API for constructing instances of PagingData.

#### PagingData: 

a component that connects the ViewModel ayer to the UI

### DataStore

a data storage solution that allows you to store key-value paris or typed objects with protocol buffers. 

#### Two Implementations

- __Preferences DataStore__: stores and accesses data using keys. This implementation does not require a predefined schema, and it does not provide type safety. like redis.

- __Proto DataStore__: stores data as instances of a custom data type. This implementation requires you to define a schema using protocol buffers, but it provides type safety.

### WorkManager (Schedule tasks)

an API taht makes it easy to schedule reliable, asynchronous tasks that are expected to run even if the app exists or the device restarts.

#### Features

1. Work constraints: give a constraint when the schedules task is executed (e.g., when wifi is available)
2. Robust Scheduling: schedule one-time or repeatedly
3. Flexible retry Policy: retry policy and exponential backoff policy.
4. Work Chaining: chain individual work task together 

## Process

System-initiated process death: will remain running until it is no longer needed and the system needs to reclaim its memory for use by other applications. 

IPC/Binder: allows applictions to communicate with each other and it is the base of several important mechanisms in the Android environment.

Binder Transaction: a message exchanged with Binder

## App Components

the essential building blocks of an Android app.

Each component is an entry point through which the system or a user can enter your app. Some components depend on others.

### activities

the entry point for interacting with the user.
It represents a single screen with a user interface.
ex) one activity: a screen to show all emails as a list
    two activity: a screen to compose an email

Although the activities work together to form a cohesive user experience in the email app, __each one is independent of the others. this is because app can start any one of acticities__.

inherit Activity class when you need to create a class for an activity.

#### permissions

control which apps can start a particular activity. 

parent and child activities must have the same permission for the parent lauch the child activiy.

#### lifecycles

should properly handle lifecycles to avoid your app crash when interruption or changing orientation (landscape/portrait), loosing the user state when coming back, consuming system resource while the activity is not active.

- onCreate: when the system creates your activity. 

any initialization is required for your activity. 

- onStart: when the activity becomes visible to the user.

- onResume: just before the activity starts interacting with the user. 

At this point, the activity is at the top of the activity stack, and captures all user input.

you implement the core functinality of this activity here.

- onPause: when the activity loses focus and enters a Paused state.

- onStop:  when the activity is no longer visible to the user.

- onRestart(): when an activity in the Stopped state is about to restart. onRestart() restores the state of the activity from the time that it was stopped.

- onDestroy: The system invokes this callback before an activity is destroyed.

diff btw onResume and onRestart: 

onResume: the next state of onPause

onRestart: the next state of onStop

#### saving and restoring transient UI state

default behavior of system:

- the system destroys the activity by default when such a configuration change occurs, wiping away any UI state stored in the activity instance. (configuration change)

- the system may destroy your application’s process while the user is away and your activity is stopped. (memory pressure)

you need to keep UI state using a combination of ViewModel, onSaveInstanceState(), and/or local storage.

Deciding how to combine these options depends on the complexity of your UI data, use cases for your app, and consideration of speed of retrieval versus memory usage

#### configuration changes

ex) change btw protrait and landscape orientations, language or input device.

when this happens, the activity is destoyed and recreated. 


#### lifecycle-aware components

a class can monitor the component's lifecycle status by adding annottions to its methods. you can acheive the decoupling as much as possible (don't need to inject the dependency into the lifecycle method in your activity)

w/o this, your activity class can easy become spagetti code with different logic. also, there's no guarantee that the component starts before the activity or fragment is stopped. This is especially true if we need to perform a long-running operation, such as some configuration check in onStart(). This can cause a race condition where the onStop() method finishes before the onStart(), keeping the component alive longer than it's needed.

#### Tasks and Back Stack

- task: a collection of activities that users interact with when performing a certain job.

- back stack: a stack where those activities are arranged 

ex)
  top activity on the back stack: the current activity the user focus
    - if the user click the back button, the activity poped off and deleted from the stack.

task can be multiple. simply, when a user start a new task or go to the home screen, via Home button, the current task is moved to the background. 

 Multiple tasks can be held in the background at once. However, if the user is running many background tasks at the same time, the system might begin destroying background activities in order to recover memory, causing the activity states to be lost.

one ativity might be instantiated mutliple times on teh same stack becasue the activities in the back stack are never rearranged. but you can modify this behavior such as you can restrict not to instantiate more than once. see next section.

#### managing tasks

you can arrange a task with <activity> manifest element and with flags in the intent that you pass to startActivity().

but __Most apps should not interrupt the default behavior for activities and tasks__. If you determine that it's necessary for your activity to modify the default behaviors, use caution and be sure to test the usability of the activity during launch and when navigating back to it from other activities and tasks with the Back button. Be sure to test for navigation behaviors that might conflict with the user's expected behavior.

## processes and app lifecycle.

every Android application runs in its own Linux process.

An unusual and fundamental feature of Android is that an application process's lifetime is not directly controlled by the application itself. instead, it is determined by the following factors:

1. the parts of the application that the system knows are running
2. how important these things are to the user
3. how much overall memory is available in the system

Not using these components (e.g., Activity, SErvice, and BroadcastReceiver) correctly can result in the system killing the application's process while it is doing important work.

## Parcelables and Bundles

objects that are used to (send/receive objects between processe, activities, and so on):

- IPC/Binder transaction
- activities with intents
- to store transient state across configuration changes. 

## Loaders

deprecated. use ViewModels and LiveData instead.

### services

general-purpose entry point for keeping an app running in the background for all kinds of reasons.

For example, a service might play music in the background while the user is in a different app, or it might fetch data over the network without blocking user interaction with an activity.

activities can start services.

two types of services

#### Started Services

tell the system to keep them running until their work is completed.

inherit Service class when you need to create a class for an activity.

##### user-awared started service

ex) music playback 

##### user-unawared started service

ex) regular background service that user does not care 

#### Bound Services

run because some other app (or the system) has said that it wants to make use of the service.

### content providers

manages a shared set of app data that you can store in the file system, in a SQLite database, on the web, or on any other persistent storage location that your app can access.

### broadcast receivers

a component that enables the system to deliver events to the app outside of a regular user flow.

ex) an app can schedule an alarm to post a notification to tell the user about an upcoming event... and by delivering that alarm to a BroadcastReceiver of the app, there is no need for the app to remain running until the alarm goes off.

## Intent

messengers that request an action from other components to activate another component.

only activate activities, services, broadcast receivers

also used when return a result from activated component.

If you use an intent to start a Service, ensure that your app is secure by using an explicit intent. Using an implicit intent to start a service is a security hazard because you cannot be certain what service will respond to the intent, and the user cannot see which service starts. __why insecure?__.

### Types

#### explicit intent

defines a message to activate a specific component. usually, a component in __your own app__.

supply the target app's package name or a fully-qualified component class name. 

#### implicit intent

defines a message to activate a specific type of component. esp general action type so that allows a component from __another app__ to handle.

if multiple possible components available with intent filter, the system displays a dialog so the user can pick which app to use.

it is useful when your app cannot perform the action, but other apps probably can and you'd like the user to pick which app to use.

- sending implicit intent: you app send an implicit intent and other app can receive the intent

ex) a user on your app want to share the content with different app to send the content (e.g., LINE with sharing)

- receiving implicit intent: 

### Pending Intent

wrapper object around Intent object.

grant permission to a foreign application to use the contained intent as if it were executed from your app's own process.

ex) Notification (the Android system's executes this intent)
    App Widget (Home screen app execute the Intent) 
    AlarmManager

use PendingIntent

### Intent Resolution

how system search for the best activity for the implicit intent it received by comparing it to intent filters based on the 3 aspects: Action, Data, Category

#### Action

must match one of the actions listed in intent-filters

if the implicit intent does not specify the action type, it pass all component which contains at least one action !!

#### Category

every category in teh intent must match a category in the filter. 

ex) 
  Intent: Category A and B

  Component:
    - intent-filters: Category A => X (failed)

  Component: 
    - intent-filters: A and B => OK (passed)

reverse is not necessary

ex)
  Component: 
    - intent-filters: A and B and C => OK (passed)

#### Data ??

- URI comparison between intent filter and implicit filter. 

ex)
  Intent: <scheme>://<host>:<port>/<path>

  Component:
    - intent-filters: <scheme> => OK

  Component: 
    - intent-filters: 


## ContentResolver

activate content providers.


## manifect file

a file to make sure the app component exisst

- activities, services, and content providers: you must define it in manifest file. Otherwise it never run. 

- broadcast receivers: can be declared in the manifest or create dynamically in code as BroadcastReceiver and register with the system by calling registerReceiver()

### intent filters

filter the type of itnents that the component would like to receive.  

another app ==== intent (ACTION_SEND) ===> your activity

intent filters only work with implicit intent. (NOT with explicit intent: it is always sent to the component regardless of the intent filters)

define intents for each unique job it can do. don't couple diffferent logic/job task together.

ex) one activity (image gallary)
    - filter1: action to view an image
    - filter2: action to edit an image

### app requirements

all android devices don't have the same feature and capabilities so you need to define your app requirement and prevent your app from installed on the devices does not meet requirements.

## app resources 

like images, sudio files, animations, menus, sytles, colors, and layout of activity user interfaces with xml files.

each resource has a unique integer ID

organized your resourced based on the different device configurations such as languages

## qualifiers

a short string that you include in the name of your resource directories to define the device configuration for which resources should be used. 

ex) portrait orientation: use that resource 
    landscap orientation: use this resource 

## Services (In Depth)
   
an application component that can perform long-running operations in the background. 
   
no user interface.
   
a service does not create a separate thread automaitcally. this is your responsibility. Otherwise, you get Application Not Responding (ANR) error when blocking operation on the main thread.
   
### Types
   
#### Foreground
   
a service which is noticable to users. 
   
must display notification to users.
   
continue working even if users are not interacting with the app. 
   
__most of the time, you should use WorkManager (scheduled task manager) to manage the foreground task___.
   
#### Background
   
a service which is not noticable to users.
   
there are some restriction when running a background service. for example, API Level 26 or higher, a background service cannot access the location of user. so you should use __WorkManager__ for bacground service also.
   
#### Bound 
   
a service which is bound to an app component.
   
the bound app component can interact with the service such as send a request, receive the result, or IPC. 
   
the lifecyle of the service is the same as the bound app component. when the component is destroyed, the service also destroyed.
   
### Service vs Thread
   
__services__: 
   
   - by default, run at the main thread, but you can run the service at different thread if your service is intensive/blocking operation.
   - use when you need to run a service at the backround while users are not interacting with the app.
   
__thread__:
   
   - run rather than the main thread.
   - only run while its corrsponding app is active.
   

   
## Kotlin

a programming lanaguage which Kotlin Application Deployment is faster to compile, lightweight, and prevents applications from increasing size. Any chunk of code written in Kotlin is much smaller compared to Java, as it is less verbose and less code means fewer bugs. Kotlin compiles the code to a bytecode which can be executed in the JVM.

### App-hopping behavior

an interruption by a phone call or notification and resume the process of the original app, and it is developer's responsibility to handle these flows correctly.

### resource-constraint

OS might kill some app processes to make room for new ones at any time.

## Android
   
## UI Thread (Main Thread)

a thread to run your app.
   
__main-safety__: it doesn't block UI updates on the main thread.
   
## Annotation
   
### @WorkerThread

denotes that the target method/class should be called on a background thread. Otherwise, it throws the exception. 
   
__you still need to do threading such as creating corouting manually. this annotation does not create a new thread automatically.__
   
### @UIThread
   
denotes that the target method/class should be called on a main thread. Otherwise, it throws the exception. 
      
## Coroutines 

a library that enables yo to write asynchronous code.

kotlin itsself does not have _async_ and _await_ keyword, but provide _suspend_ to accomplish the async. 
   
_kotlinx.coroutines_ (a rich library for coroutine) provide a number of high-level coroutine-enabled primitives including _launch_, _async_ and others.
   
you can define its scopes when your coroutines should run:

- ViewModelScope: the scope is the same as ViewModel. the coroutine is cancel when the ViewModel is cleared.
- Lifecycle: the scope is te same as Lifecycle. teh corutine is cancel when the Lifecycle is destroyed.
- repeatOnLifecycle: start your async code and cancel it at a certain lifecycle state (e.g., start your async at STARTED and cancel it at STOPPED)
 - every time the lifecycle enter one state and leaves other state
- whenCreated, WhenStartd, WhenResume: to suspend asycn execution unless the Lifecycle is in a certain state.

prefer repeatOnlifeCycle (cancel) over whenX or launchWhenX (suspension) since supension cause its state active in the background, potentially emitting new items and wasting resources.

structured concurrency: how to handle related multiple coroutine (parent and child) with a scope

- need to properly manage (start and cancel) multiple coroutine related each other with a scope. if the scope is cleared, we need to cancel the all related coroutines.
- see this: https://elizarov.medium.com/structured-concurrency-722d765aa952

use 'emit(..)' to return the result from async code

__dispatcher__: decide which thread or threads the corrsponding coroutine uses to execute its execution. for example, it can confine a given execution to be run on a specific thread, send it to thread pool, or run on the main thread.
   
Because coroutines can easily switch threads at any time and pass results back to the original thread, it's a good idea to start UI-related coroutines on the Main thread.
  
 
## Android UI Features

### pull-to-refresh

pull a ui component and it cause to refresh the items.

you can use a library for the feature: https://stackoverflow.com/questions/4583484/how-to-implement-android-pull-to-refresh

### Deep Link
   
   in nutshell, guide users to an activity on the deeper hierarchy of your app without openinig home page and navigate to the page. 

   a concept that help users navigate between the web and applications. 

   a url which navigate users directly to the specific content in applications.
   
   ref: https://www.youtube.com/watch?v=XJgPIeolJu8&ab_channel=AndroidDevelopers
   
### Android App Link 
   
   allow an app to designate itself as the default handler of application domain or URL. (API Level 23 or higher)

   
## Security

- Using an intent filter isn't a secure way to prevent other apps from starting your components. Although intent filters restrict a component to respond to only certain kinds of implicit intents, another app can potentially start your app component by using an explicit intent if the developer determines your component names. If it's important that only your own app is able to start one of your components, do not declare intent filters in your manifest. Instead, set the exported attribute to "false" for that component. Similarly, to avoid inadvertently running a different app's Service, always use an explicit intent to start your own service.

## Performance
  
### Goal
  
  1. use power sparingly
  2. start up quickly
  3. respond quickly to user interaction
  
### Layout
   
#### optimize layout hierarchy

- __nested LinearLayout__ causes performance issue (slow down) to make it flatten with RelativeLayout.
   
- __layout_weight in teh LinearLayout__ slow down the speed of measurement.
   
- __use Lint__ to search for possible view hierarchy optimizations. 

- use __ConstraintsLayout__ rather than Relative/LinearLayout to avoid nested deep view tree
   

   
