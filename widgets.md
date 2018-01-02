# Fantasy POS - Widgets

The widgets module is a globally accessible module, which serves as a manager for all widgets, available under appDir/widgets/. This module has the ability to automatically invoke widgets defined in your view. You can also use this module to programmatically invoke html widgets added dynamically to your view. This module is using ammo.app and the node convention. It also utilizes globalEvents to automatically detect html widgets.

## Purpose

The widgets module was developed with several goals in mind:

1. Has the ability to detect and automatically invoke html-defined widgets, without any code required from the client coder.
2. When automatically detecting widgets, the module must be able to correctly decide when to load widgets based on their showOn property.
2. Contains in an encapsulated way all globally available widgets
3. Provide a small event system, via globalEvents, for interacting with widgets

## Usage

Widgets are UI functionalities, encapsulated in html divs. They contain some attributes which define their behavior. Fantasy POS currently supports a number of widgets, including:

- dropdown - toggleable dropdown widget
- header - navigation widget which displays the name for the current view plus optional navigation buttons.
- loader - widget for visualizing that something is being loaded
- multiselect - multiselect widget
- navigation - navigation lists available in different layouts
- reorderable - reorderable list with delete option

Typically, a widget defined in your html markup looks like this:

```html
<div
    class="widget"
    data-widget="header"
    data-has-navigation="true"
    data-title="Settings"
    data-show-on="loading"
></div>
```

Another simpler example:

```html
<div class="widget" data-widget="loader" data-show-on="loading"></div>
```

These two `html` widgets will be automatically detected and invoked by the widgets module. Notice the `data-show-on` attribute. It says that both widgets will be invoked when the view loading global event is dispatched.

Currently, the convention for the widgets' showOn attribute is to add this attribute if you wish your widget to be automatically loaded when the view is being loaded. Or, do not use this attribute at all if you wish your widget to be loaded on the global view ready event.

Generally, you can use widgets in two ways:

- intercepting widget ready events so you can dispatch a widget change action
- Programmatically invoke dynamically added widgets

##### Intercepting widget ready to dispatch a widget change action

```javascript
/* globals widgets */

// get widgets' events
const widgetEvents = widgets.getNodes('events');

widgetEvents.onWidgetReady('header', () => {
    widgetEvents.triggerWidgetChange('header', {
        type: 'update-title',
        title: 'Hey, this is the new title'
    });
});
```

The code above will intercept when the ready event for widget 'header' is triggered. Then, the callback will trigger a change event for widget 'header'. This change event will be of type 'update-title' and will contain also the new title, which widget 'header' must use to update its title.

##### Programmatically invoke dynamically added widgets

Let's say that you need to dynamically add a widget to your page after your view/module is ready. For example, you wish to display a modal with a multiselect widget in it. This can be easily done by initializing manually the required widget. Here's how to do it:

First, you need to append your html widget to the DOM. Then in your javascript, you need to do the following:

```javascript
/* globals widgets */

// retrieve factory for multiselect widget
const multiselectWidget = widgets.getNode('widgets', 'multiselect');

// get widget DOM node
const domWidget = ammo.select('[data-widget="multiselect"]').get();

// initialize the widget
multiselectWidget(domWidget);
```

`Note:` Currently, you need to pass the DOM node for the widget you wish to manually initialize. Optionally, you can pass a props object in case you wish to customize the widget upon initialization.

## How to extend widgets

In order to add a new widget, you need to add a new path under appDir/widgets, named after the new widget. This path should contain the widget's javascript file and an optional scss file for the widget's styles.

## Best Practices

Keep widget's functionality compact. Ideally, widgets should encapsulate trivial UI functionality like toggling dropdowns or deleting DOM nodes.

Since all widgets utilize ammo.app and the node convention, it is advisable to use the same approach when adding new widgets.
