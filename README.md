# Component Lifecycle
Android is designed to empower users and let them use apps in a intuitive way so we should know how to manage component lifecycles. A component can be an Activity, Fragment, Service, Application itself and even underlying process. The component has lifecycle, during which it transations through states. Whenever transation happens, the system notifies you via lifecycle callback method.

## Activity 
- onCreate() = called when an activity first created; when a user opens the app
- onStart() = called when an activity becomes visible to the user
- onResume() = called when a user starts interacting with the application
- onPause() = when activity is paused and receiving other activities in onResume method
- onStop() = called when activity is no longer visible to the user; when a user minimizes the app
- onDestroy() = called when activity destroyed; when the users clear the application stack
- onRestart() = called when activity restarts

### Scenario 1: App is finished and restarted
Triggered by:
 - Pressed Back Button, or 
 - Activity.finish() method is called
 
1. [App started] - onCreate(null) -> onStart() -> onResume()
2. [Back pressed] - onPause() -> onStop() -> onDestroy
3. [App Restarted] - onCreate(null) -> onStart -> onResume()
 
 Managing state: 
 - onSaveInstanceState is not called (since the activity is finished, you don't need to save state)
 - onCreate doesn't have a Bundle when the app is reopened, because the activity was finished and the state doesn't need to be restored

### Scenario 2: User navigates away
Triggered by:
- Pressed Home Button
- Switches to another app (via Overview menu, from a notification, accepting a call, etc)

1. [App started] - onCreate(null) -> onStart() -> onResume()
2. [Home pressed] - onPause() -> onStop() -> onSaveInstanceState()
3. [App restarted] - onCreated() -> onStart() -> onResume()

This scenario the system will instantly stop the activity, but won't immediately finish it.

Managing state: 
When activity entes the Stopped state, the system uses onSaveInstanceState to save the app state in case the system kills the app's process later on

### Scenario 3: Configuration changes
Triggered by:
- Configuration changes, like rotation
- User resizer the window in multi-window mode

1. [App started] - onCreate(null) -> onStart() -> onResume()
2. [Device rotated] - onPause() -> onStop() -> onSaveInstanceState() -> onDestroy() -> onCreate(Bundle) -> onStart() -> onRestoreInstanceState() -> onResume()

Managing state:
- Activity is completely destroyed, but the state is saved and restored for the new instance

### Scenario 4: App is paused by the system
Triggered by:
- Enabling multi window mode and losing the focus 
- Another app partially covers the running app (a purchase dialog, a runtime permission dialog, a third-party login dialog)
- An intent chooser appears, such as a share dialog

1. [App started] - onCreate(null) -> onStart() -> onResume()
2. [Multi-window started] - onPause()
3. [Focus comes back to app] - onResume()

## Multiple Activities
### Scenario 1 - Back Stack: Navigating between activities 
1. [Activity 1 starts]
- (Activity 1) - onCreate() -> onStart() -> onResume()
  
2. [Activity 2 starts]
- (Activity 1) - onPause() -> onStop() -> onSaveInstanceState()
- (Activity 2) - onCreate() -> onStart() -> onCreate(null) -> onStart() -> onResume()
  
3. [Back pressed] 
- (Activity 1) - onRestart() -> onStart() -> onResume()
- (Activity 2) - onPause() -> onStop() -> onDestroy()
  
4. [Back pressed]
- (Activity 1) - onPause() -> onStop() -> onDestory()
  
Managing state:
- Note that onSaveInstanceState is called, but onRestoreInstanceState is not. if there is a configuration change when the second activity is active, the first activity will be destroyed and recreated only when it's back in focus. That's why saving an instance of the state is important.

### Scenario 2 - Back Stack: Activities in the back stack with configuration changes
1. [Activity 1 is in the back stack and Activity 2 is in the foreground]
- (Activity 1) - onStop() -> onSaveInstanceState()
- (Activity 2) - onStart() -> onResume()
  
2. [Device rotated]
- (Activity 2) - onPause() -> onStop() -> onSaveInsatanceState() -> onDestroy() -> onCreate(Bundle) -> onStart() -> onRestoreInstanceState() -> onResume()
  
3. [Back pressed] 
- (Activity 1) - onDestroy() -> onCreate(Bundle) -> onStart() -> onRestoreInstanceState() -> onResume()
- (Activity 2) - onPause() -> onStop() -> onDestroy()
  
Managing state:
- All activities in the stack need to restore state after a configuration change to recreate UI.

### Scenario 3 - Back Stack: App's process is killed
1. [Activity 1 is in the back stack and Activity 2 is in the foreground]
- (Activity 1) - onStop() -> onSaveInstanceState()
- (Activity 2) - onStart() -> onResume()
  
2. [Home pressed]
- (Activity 2) - onPause() -> onStop() -> onSaveInstanceState()
  
3. [System kills process]
4. [User opens app again]
- (Activity 2) - onCreate(Bundle) -> onStart() -> onRestoreInstanceState() -> onResume()
  
5. [Back pressed] 
- (Activity 1) - onCreate(Bundle) -> onStart() -> onRestoreInstanceState() -> onResume()
- (Activity 2) - onPause() -> onStop() -> onDestroy()
  
Managing state:
- Note that the state of the full back stack is saved but, in order to efficiently use resources, activities are only restored when they are recreated.

## Fragment
When fragment come up on the screen:
- onAttach() = to know that our fragment has been attached to an activity. We are passing the activity that will host our fragment
- onCreate() = fragment instance initializes, just after the onAttach where fragment attaches to the host activity
- onCreateView() = time for the fragment to draw its user interface for the first time.
- onActivityCreated() = activity completes its onCreate() methid
- onStart() = this method called when a fragment is visible
- onResume() = fragment is visible and following the user to interact with it. Fragment resumes only after activity resumes.

When fragment goes out off the screen:
- onPause() = when fragment is not allowing the user to interact; the fragment will get change with other fragment or it gets removed from activity or fragment’s activity called a pause.
- onStop() = when fragment is no longer visible; the fragment will get change with other fragment or it gets removed from activity or fragment’s activity called stop.
- onDestroyView() = when view and related resources created in onCreateView are removed from the activity's view hirarchy and destroyed.
- onDestroy() = when fragment does its final clean up
- onDetach() = when fragment is detached from its host activity

### Scenario 1 - Activity with Fragments starts and finishes
1. [App started]
- (Activity 1) - onCreate()
- (Fragment 1) - onAttach() -> onCreate() -> onCreateView(null) -> onActivityCreated(null)
- (Activity 1 and Fragment 1) - onStart()
- (Activity 1 and Fragment 1) - onResume()

2. [Back pressed]
- (Activity 1 and Fragment 1) - onPause()
- (Activity 1 and Fragment 1) - onStop()
- (Activity 1) - onDestroy()
- (Fragment 1) - onDestroyView() -> onDestroy() -> onDetach()

### Scenario 2 - Activity with Fragments is rotated
1. [Activity and Fragment are in RESUMED state]
2. [Device is rotated]
- (Activity 1 and Fragment 1) - onPause()
- (Activity 1 and Fragment 1) - onStop()
- (Activity 1 and Fragment 1) - onSaveInstanceState()
- (Activity 1) - onDestroy() 
- (Fragment 1) - onDestroyView() -> onDestroy() -> onDetach()
- (Activity 1) - onCreate(Bundle)
- (Fragment 1) - onAttach() -> onCreate(Bundle) -> onCreateView(Bundle)
- (Fragment 1) - onActivityCreated(Bundle)
- (Activity 1 and Fragment 1) - onStart()
- (Activity 1) - onRestoreInstanceState()
- (Activity 1 and Fragment 1) - onResume()

State management:
- Fragment state is saved and restored in very similar fashion to activity state. The difference is that there's no onRestoreInstanceState in fragments, but the Bundle is available in the fragment's onCreate, onCreateView and onActivityCreated

### Scenario 3 - Activity with retained fragment  is rotated
1. [Activity and Fragment are in RESUMED state]
2. [Device is rotated]
- (Activity 1 and Fragment 1) - onPause()
- (Activity 1 and Fragment 1) - onStop()
- (Activity 1 and Fragment 1) - onSaveInstanceState()
- (Activity 1) - onDestroy() 
- (Fragment 1) - onDestroyView() -> onDetach()
- (Activity 1) - onCreate(Bundle)
- (Fragment 1) - onAttach() -> onCreateView(Bundle)
- (Fragment 1) - onActivityCreated(Bundle)
- (Activity 1 and Fragment 1) - onStart()
- (Activity 1) - onRestoreInstanceState()
- (Activity 1 and Fragment 1) - onResume()

State management:
- Fragment is not destroyed nor created after the rotation because the same fragment instance is used after the activity is recreated. State bundle is still available in onActivityCreated. Using retained fragments is not recommended unless they are used to store data across configuration changes (in a non-UI fragment)
