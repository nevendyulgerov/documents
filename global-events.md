# Fantasy POS - Global Events

The globalEvents module is a globally accessible module, through which you can dispatch and intercept global events. You can use these events to notify the app that a view is being loaded, a widget is ready and much more. This module is using the pub/sub design pattern.

## Purpose

The globalEvents module enable you to load functionality from different locations (modules, widgets, scripts) into a single view, while maintaining separation of concept.

Originally, this module was developed with the goal to provide the necessary bridge, required to de-couple module functionality from UI functionality. It has evolved into a system through which you can elegantly connect different parts of the application.

Currently via globalEvents, the app can automatically load widgets on modules' views. An example of this is the widget 'loader' which can be easily initialized via globalEvents to provide visual feedback that your module is being loaded while its data is being pulled. More on how widgets work, can be found in their own documentation.

## Usage

Currently, you can use globalEvents to notify/intercept that:

- a view is loading or is ready
- a module is ready or is changing
- a widget is ready or is changing

##### View is loading

```javascript
/* globals globalEvents */

// dispatch event for view loading
globalEvents.dispatchViewLoading();

// intercept event for view loading
globalEvents.interceptViewLoading(() => {
    // your callback code...
});
```

`Note:`: The .dispatchViewLoading() action should be invoked inside the module controlling the current view, before any work to display the current view is done.

`Note:` The .interceptViewLoading() action will execute your callback when dispatchViewLoading is invoked.

##### View is ready

```javascript
/* globals globalEvents */

// dispatch event for view loading
globalEvents.dispatchViewReady();

// intercept event for view loading
globalEvents.interceptViewReady(() => {
    // your callback code...
});
```

`Note:`: The .dispatchViewReady() action should be invoked inside the module controlling the current view, after all work to display the current view is done.

`Note:` The .interceptViewReady() action will execute your callback when dispatchViewLoading is invoked.

`Note:` Since in Fantasy POS, the routes are designed to load a single view, you need to pass a single parameter to .interceptViewReady() - the callback for that global event. No view names are required to dispatch/intercept events for view loading.

##### Module is ready

```javascript
/* globals globalEvents */

// define module name
const moduleName = 'accounts';

// dispatch event for module ready
globalEvents.dispatchModuleReady(moduleName);

// intercept event for module ready
globalEvents.interceptModuleReady(moduleName, () => {
    // your callback code...
});
```

`Note:`: The .dispatchModuleReady() action should be invoked inside the module controlling the current view, after all work to display the current view is done.

`Note:` The .interceptModuleReady() action will execute your callback when dispatchModuleReady with the provided module is invoked.

`Note:` Since there are many modules in Fantasy POS, you need to specify which module you wish to dispatch and intercept ready events for via the moduleName param.

##### Module is changing
Hooking to this event is a good idea when you want to remotely ask a module for updates. This usually happens when at some stage your code needs to communicate with modules.

```javascript
/* globals globalEvents */

// define module name
const moduleName = 'accounts';

// define module state for change
const moduleState = {type: 'empty-accounts'};

// dispatch event for module change with moduleName and moduleState
globalEvents.dispatchModuleChange(moduleName, moduleState);

// intercept event for module change
globalEvents.interceptModuleChange(moduleName, (event, options) => {
    switch (options.type) {
        case 'empty-accounts':
            // handle action
        default:
            // handle default action
    }
});
```

`Note:` The .dispatchModuleChange(moduleName, moduleState) takes two parameters - the module name you want to update and the state of the update. State objects must contain a type property, which should define the type of change that the module should execute. The .dispatchModuleChange() can be invoked from anywhere.

`Note:` The .interceptModuleChange() hook must be invoked inside the changing module's definition in order to take advantage of its internal methods.

##### Widget is ready

```javascript
/* globals globalEvents */

// define widget name
const widgetName = 'header';

// dispatch event for widget ready
globalEvents.dispatchWidgetReady(widgetName);

// intercept event for widget ready
globalEvents.interceptWidgetReady(widgetName, () => {
    // your callback code...
});
```

`Note:`: The .dispatchWidgetReady() action should be invoked inside the widgets code, after all work to display the widget is done.

`Note:` The .interceptWidgetReady() action will execute your callback when dispatchWidgetReady for the provided widget is invoked.

`Note:` Since there are many widgets in Fantasy POS, you need to specify which widget you wish to dispatch and intercept events for via the widgetName param.

##### Widget is changing
Hooking to this event is a good idea when you want to remotely ask a widget for updates. This usually happens when at some stage your code needs to communicate with widgets.

```javascript
/* globals globalEvents */

// define widget name
const widgetName = 'header';

// define widget state for change
const widgetState = {
    type: 'update-title'
    title: 'Hey, this is the new title'
};

// dispatch event for widget changing with widgetName and widgetState
globalEvents.dispatchWidgetChange(widgetName, widgetState);

// intercept event for module ready
globalEvents.interceptWidgetChange(widgetName, (event, options) => {
    switch (options.type) {
        case 'update-title':
            // handle action
        default:
            // handle default action
    }
});
```

`Note:` The .dispatchWidgetChange(widgetName, widgetState) takes two parameters - the widget name you want to update and the state of the update. State objects must contain a type property, which should define the type of change that the widget should execute. The .dispatchWidgetChange() can be invoked from anywhere.

`Note:` The .interceptWidgetChange() hook must be invoked inside the widgets's definition in order to take advantage of its internal methods.

## How to extend globalEvents

Module globalEvents resides in appDir/scripts/global-events.js. In order to extend the module you need to perform 4 steps per update:

1. Define a constant for your event, using a name in all caps, like the existing event name constants.
2. Add a switch case interceptor for the new event in dispatchEvent, using the existing convention
3. Add a switch case interceptor for the new event in interceptEvent, using the existing convention
4. Define the intercept/dispatch public methods which communicate with the private dispatchEvent/interceptEvent methods, using the existing convention

## Best Practices

###### Keep the number of supported events to a bare MINIMUM.

globalEvents is designed to maintain a small but globally applicable number of events. So, when wondering whether or not to extend globalEvents ask yourself the following question. Is this event usable by ALL other parts of the application? If the answer is no, then use a local strategy, rather than extending globalEvents with a specialized event.

Remember, globalEvents provide app-critical, rather than module-specific events.

