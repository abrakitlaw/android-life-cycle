# Component Lifecycle
Android is designed to empower users and let them use apps in a intuitive way so we should know how to manage component lifecycles. A component can be an Activity, Fragment, Service, Application itself and even underlying process. The component has lifecycle, during which it transations through states. Whenever transation happens, the system notifies you via lifecycle callback method.

## Activity 
Scenario 1: App is finished and restarted
Triggered by:
 - Pressed Back Button, or 
 - Activity.finish() method is called
 
[App started] - onCreate(null) -> onStart() -> onResume()
[Back pressed] - onPause() -> onStop() -> onDestroy
[App Restarted] - onCreate(null) -> onStart -> onResume()
 
 Managing state: 
 - onSaveInstanceState is not called (since the activity is finished, you don't need to save state)
 - onCreate doesn't have a Bundle when the app is reopened, because the activity was finished and the state doesn't need to be restored

Scenario 2: User navigates away
Triggered by:
- Pressed Home Button
- Switches to another app (via Overview menu, from a notification, accepting a call, etc)

[App started] - onCreate(null) -> onStart() -> onResume()
[Home pressed] - onPause() -> onStop() -> onSaveInstanceState()
[App restarted] - onCreated() -> onStart() -> onResume()

This scenario the system will instantly stop the activity, but won't immediately finish it.

Managing state: 
When activity entes the Stopped state, the system uses onSaveInstanceState to save the app state in case the system kills the app's process later on

Scenario 3: Configuration changes
Triggered by:
- Configuration changes, like rotation
- User resizer the window in multi-window mode

[App started] - onCreate(null) -> onStart() -> onResume()
[Device rotated] - onPause() -> onStop() -> onSaveInstanceState() -> onDestroy() -> onCreate(Bundle) -> onStart() -> onRestoreInstanceState() -> onResume()

Managing state:
- Activity is completely destroyed, but the state is saved and restored for the new instance

Scenario 4: App is paused by the system
Triggered by:
- Enabling multi window mode and losing the focus 
- Another app partially covers the running app (a purchase dialog, a runtime permission dialog, a third-party login dialog)
- An intent chooser appears, such as a share dialog

[App started] - onCreate(null) -> onStart() -> onResume()
[Multi-window started] - onPause()
[Focus comes back to app] - onResume()

## Multiple Activities
Scenario 1 - Back Stack: Navigating between activities 
[Activity 1 starts]
  (Activity 1) - onCreate() -> onStart() -> onResume()
  
[Activity 2 starts]
  (Activity 1) - onPause() -> onStop() -> onSaveInstanceState()
  (Activity 2) - onCreate() -> onStart() -> onCreate(null) -> onStart() -> onResume()
  
[Back pressed] 
  (Activity 1) - onRestart() -> onStart() -> onResume()
  (Activity 2) - onPause() -> onStop() -> onDestroy()
  
[Back pressed]
  (Activity 1) - onPause() -> onStop() -> onDestory()
  
Managing state:
- Note that onSaveInstanceState is called, but onRestoreInstanceState is not. if there is a configuration change when the second activity is active, the first activity will be destroyed and recreated only when it's back in focus. That's why saving an instance of the state is important.

Scenario 2 - Back Stack: Activities in the back stack with configuration changes
[Activity 1 is in the back stack and Activity 2 is in the foreground]
  (Activity 1) - onStop() -> onSaveInstanceState()
  (Activity 2) - onStart() -> onResume()
  
[Device rotated]
  (Activity 2) - onPause() -> onStop() -> onSaveInsatanceState() -> onDestroy() -> onCreate(Bundle) -> onStart() -> onRestoreInstanceState() -> onResume()
  
[Back pressed] 
  (Activity 1) - onDestroy() -> onCreate(Bundle) -> onStart() -> onRestoreInstanceState() -> onResume()
  (Activity 2) - onPause() -> onStop() -> onDestroy()
  
Managing state:
- All activities in the stack need to restore state after a configuration change to recreate UI.

Scenario 3 - Back Stack: App's process is killed
[Activity 1 is in the back stack and Activity 2 is in the foreground]
  (Activity 1) - onStop() -> onSaveInstanceState()
  (Activity 2) - onStart() -> onResume()
  
[Home pressed]
  (Activity 2) - onPause() -> onStop() -> onSaveInstanceState()
  
[System kills process]
[User opens app again]
  (Activity 2) - onCreate(Bundle) -> onStart() -> onRestoreInstanceState() -> onResume()
  
[Back pressed] 
  (Activity 1) - onCreate(Bundle) -> onStart() -> onRestoreInstanceState() -> onResume()
  (Activity 2) - onPause() -> onStop() -> onDestroy()
  
Managing state:
- Note that the state of the full back stack is saved but, in order to efficiently use resources, activities are only restored when they are recreated.
