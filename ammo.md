# Fantasy POS - Ammo

Ammo is a core library, used extensively in the front-end code of Fantasy POS. It offers a number of javascript utilities for common development tasks including an app construct, promise API, DOM manipulation API, buffering system, persistent storage API and many more. You can use Ammo to build lean functionality within the Fantasy POS app. This library has zero dependencies and utilizes ES6 code.

## Purpose

Ammo was developed with the goal to make heavy-in-size and heavy-upon-initialization libraries like jQuery and Dust obsolete. It offers common utilities for many of the available methods in jQuery, related to DOM manipulation. It exposes a stand-alone AJAX API for simple JSON-based requests. It also has a replacement for the events system available in jQuery. All of ammo's methods are light-weight. They apply minimum abstraction layers on top of standard ES6 code to achieve the required functionality. This keeps the library fast and efficient.

## Usage

Some of the commonly used Ammo utilities are:

- app() - an app construct, for building encapsulated, functional apps with manageable store
- sequence() - a recursion-based promise implementation
- select() and selectAll() - DOM-manipulation APIs

### Ammo.app

#### General examples

Ammo.app allows for the creation of elegant, encapsulated, functional apps with manageable store. This construct uses extensively the cascade and curry design patterns. Here's an example of how to use ammo.app:

```javascript
/* globals ammo */

// create local app with a 'default' schema
const app = ammo.app().schema('default');
```

This creates a local app with the following internal schema:
```javascript
const app = {

    // space for the app's props
    props: {},

    // space for the app's store
    store: {},

    // space for the app's nodes
    nodes: {

        // node family 'events'
        events: {},

        // node family 'renderers'
        renderers: {},

        / node family 'actions'
        actions: {}
    }
};
```

The initialization of the above app can be read as 'create a local ammo.app with nodes schema - events, renderers, actions'. To add some functionality to your app, you need to use the node API:

```javascript

// configure node family 'actions', then add to it a node 'init'
app.configure('actions')
    .node('init', () => {
        console.log('my app is initialized');
    });
```

The .configure('nodeFamily').node('nodeName', function) is a currying method. This will augment node family 'actions' with a node (functionality), called 'init'. After this action, your internal app schema will look like this:

```javascript
const app = {
    props: {},
    store: {},
    nodes: {
        events: {},
        renderers: {},
        actions: {
            init: () => {
                console.log('my app is initialized');
            }
        }
    }
};
```

You can chain node definitions:

```javascript
app.configure('actions')
    .node('getData', () => {
        return {data: 123};
    })
    .node('init', () => {
        console.log('my app is initialized');
    });
```


**In an ammo.app all functionality resides in nodes**

`Note:` An important take-away from the examples above is that in an ammo.app all functionality resides in nodes. Each of your methods is a node. A group of related-by-purpose nodes is called a node family. In the examples above events, renderers and actions are node families. Following this pattern, node family 'events' should contain only event handlers. Node family 'renderers' should contain only nodes concerned with the rendering of DOM nodes. Node family 'actions' should contain only nodes which expose app functionality, different from event-handling or rendering.

`Note:` By convention, all apps should contain an actions node family. In this node family, you need to have at a bare minimum an 'init' node, which will work as the app's entrypoint. Typically, the init node of an ammo.app serves as a middleware for connecting the app's nodes.

#### Utilizing the ammo.app store

Ammo.app supports the props/state convention, found in frameworks like React. This convention basically says that each app can have two separate objects - one for the app's props (read-only attributes) and one for the app's state (read/write attributes). The state is where mutable data should be stored.

Here's how to initialize an ammo.app with props/state. We will be using another feature of ammo.app - the ability to define global apps.

```javascript
/* globals ammo */

// define app's props
const props = {
    name: 'myApp',
    global: true,
    user: 'guest'
};

// define app's initial state
const state = {
    items: []
};

// create global app with props and state, and schema - events, renderers, action
const app = ammo.app(props, state).schema('default');

// you can now access your app from the global window object
// window.myApp will be your app
```

The app's props are ready only. You can only retrieve them. Here's how to do it:

```javascript

// get app
const app = window.myApp;

// get all app props
const appProps = app.getProps();

// get specific app prop
const appPropUser = app.getProps('user');
```

You can read and write to the app's state. Here's how to do it:

```javascript

// get app
const app = window.myApp;

// define some data
const newItems = [{key: 'a'}, {key: 'b'}];

// update app store
app.updateStore('items', () => newItems);
```

The .updateStore() method takes a store key and a PURE function. The function must return the new value for that store key. This new returned value will replace the last value in the app, available on the requested store key. The .updateStore() method can also provide the last available value on the requested store key:

```javascript
app.updateStore('items', lastValues => [...lastValues, ...newItems]);
```

By utilizing the available argument `lastValues` you can augment the existing data, rather than replacing it.

`Note:` In ammo.app state and store are the same thing.

`Note:` Ammo.app follows the convention that each app can have its own store, similarly to a Flux-based architectures. There can be multiple smaller apps with their respective stores in your app. This creates a nice separation of concepts in terms of data management. Your entire app can be seen as a group of smaller apps with specific purpose, each with their own self-managed store.

#### Utilizing the ammo.app store with caching

Another cool feature of ammo.app is its ability to provide seamless synchronization between your app's store and a configurable space in the browser's persistent storage facilities like localStorage. This functionality is available via the .syncWithPersistentStore() method. Here's how to take advantage of it:

```javascript
/* globals ammo */

// define app's props
const props = {
    name: 'myApp',
    global: true,
    user: 'guest',
    storeKey: 'myApp'
};

// define app's initial state
const state = {
    items: []
};

// create global app with props and state, and schema - events, renderers, action
// also synchronize the app with localStorage, using store key 'myApp'
const app = ammo.app(props, state).schema('default').syncWithPersistentStore();

// you can now access your app from the window object
// window.myApp will be your app
```

With a single method call you have enabled powerful caching for your app's state. Now, every time you perform an update via the .updateStore('storeKey', () => ['newData']) method, this data will be synchronized with the data, residing under localStorage::key('myApp'). This means that your data will persist through page reloads. With this functionality, features like maintaining state of configurable UI, become trivial.

`Note:` When you need to deal with state in an ammo.app always opt for the built-in state management facilities, rather than maintaining external objects which interact with the app via custom code. You will be rewarded with a much more elegant app structure and less worries related to state management.

#### Strong types in ammo.app store

Ammo.app can also govern the type of the data, stored in its store. This means that you can create strongly-typed apps with ammo.app. Here's an example of how to do this:

```javascript

// define props with the attribute 'strongTypes' set to true
const props = {
    strongTypes: true
};

// define a store with some strong-typed data
const store = {
    settings: {type: 'object', value: {}}
};

// initialize an app with strong types for its store with schema - events, renderers, actions
const myApp = ammo.app(props, store).schema('default');
```
To use strong types you need to pass the attribute 'strongTypes' set to true as part of the app's props. Strong types are applicable only for store data. The format in which you need to store a strong-typed item is:

```javascript
/*
[{storeKey}]: {
    type: {itemType},
    value: {initialItemValue}
}
*/
```

So, each item in your store needs to be wrapped in an object. In this object, you need to have two properties, one for the type and one for the initial value.

Once you initialize your app, you can play with the store to see the results:

```javascript

// this update will succeed, as you pass a value matching the item's type
myApp.updateStore('settings', () => {
    return {
        a: 'test',
        b: 123
    };
});

// this update will fail, as you pass a value not matching the item's type
// check the console for the exact error
myApp.updateStore('settings', () => 'abc');
```

`Note:` It is strongly recommended that you define all of your ammo.apps' stores as strong-typed stores. This will increase the resilience and predictability of your applications.

`Note:` Remember that all type checking happens at runtime. This means that strong types are mostly useful for development since they can expose potential defects related to types when the store update happens. Do not rely on strong types in production.

`Note:` The available strong types are:

- 'number' - accepts numbers which return false on NaN checks
- 'string' - accepts strings
- 'bool' - accepts boolean
- 'object' - accepts non-array, non-null objects
- 'null' - accepts null
- 'function' - accepts function
- 'array' - accepts non-object arrays
- 'undefined' - accepts undefined

`Note:` Internally, the checks for strong types rely on the type checker methods isNum, isStr, isBool, etc. These methods are directly available from ammo.

`Note:` Strong types are achieved with ES6's proxy implementation by setting a 'set' trap that checks the type of the passed data.

#### Inheritance in ammo.app

Ammo.app supports inheritance via the .inherit() method. Use this method when you want your app to inherit functionality from another ammo.app:

```javascript

// add a node 'render' to node family 'renderers'
myApp.configure('renderers')
    .node('render', (target, html) => target.innerHTML = html);

// add several nodes to node family 'actions'
myApp.configure('actions')
    .node('getData', () => [1, 2, 3, 4, 5])
    .node('filterData', data => data.filter(item => item > 3))
    .node('init', () => {
        const actions = myApp.getNodes('actions');
        const data = actions.getData();
        console.log(`myApp initialized. Data: ${actions.filterData(data)}`)
    });

// create a new app and inherit all nodes from myApp
const newApp = ammo.app().inherit(myApp);

// create another app, this time inheriting only node family 'actions'
const anotherApp = ammo.app().inherit(myApp, ['actions']);
```

As you can see you can inherit either the entire node tree of an app or just specific node families. Inheriting specific nodes from specific node families is currently not supported.

`Note:` The actual inherit operation is not memory efficient - eg. all nodes to be inherited will be recreated in memory for the inheriting app. Ammo, in general, does not interact with the prototypical chain.

#### Overwriting existing functionality in ammo.app

Overwriting in ammo.app is available via the .overwrite() curried method. Here's how:

```javascript

// overwrite myApp's node family 'renderers'
myApp.overwrite('renderers')
    .node('render', (target, html) => target.insertAdjacentHTML('beforeend', html));
```

`Note:` The .overwrite() methods works just as .configure() - it gives access to the 'node' method. Each node defined under an .overwrite() will overwrite an existing node with the same name under the same node family.

`Note:` You cannot re-define nodes using the .configure('nodeFamily').node('nodeName') API. If you attempt to overwrite existing nodes via .configure().node() no changes will be applied over the app. The only way to reassign a node is by using the .overwrite() method.

#### Calling and getting nodes in ammo.app

To get a node, use the .getNode() method. To directly call a node, use the .callNode() method:

```javascript

// store node into a variable
const filterData = myApp.getNode('actions', 'filterData');

// call the node afterwards
filterData();

// call node directly
myApp.callNode('actions', 'init');
```

`Note:` The .getNode() method is preferred when you need to invoke a node with arguments. With it, you first need to get the node and then invoke it. The .callNode() method is preferred when the node has zero arguments. The .callNode() will immediately invoke the desired node.

#### Ammo.app schemas

The .schema() method in ammo.app allows for applying a predefined schema of node families to your app.

There are several built-in schemas in ammo.app. These include:

- default   - node families -> events, renderers, actions
- app       - node families -> events, actions, common, modules, core
- module    - node families -> events, actions, templates, views
- widget    - node families -> events, actions, widgets

You can apply any of these schemas, using the .schema() method:

```javascript

// create an app with the 'module' schema
const moduleApp = ammo.app().schema('module');

// create an app with the 'widget' schema
const widgetApp = ammo.app().schema('widget');
```

`Note:` You can easily augment your app with a new node family using the .augment('nodeFamily') method.

#### Other ammo.app methods

There are several other methods available in ammo.app:

```javascript

// get all app nodes
const nodes = myApp.getNodes();

// get nodes from specific node family
const actions = myApp.getNodes('actions');

// add a single node family 'templates' to your app
myApp.augment('templates');

// add node to a node family shorthand method
myApp.addNode('templates', 'index', () => (`<div class="template">My template</div>`));

if ( myApp.nodeExists('templates', 'index') ) {
    // node 'index', under node family 'templates' exist
}

// get persistent store
const persistentStore = myApp.getPersistentStore();

// clear persistent store for app
myApp.clearPersistentStore();

// add custom schema (note that this schema will not be automatically applied to the app)
myApp.addSchema('custom', ['ui', 'actions']);

// add schema 'custom' to the app (note that this will add only the node family 'ui' to the app, since 'actions' is already defined)
myApp.schema('custom');
```

## Ammo.sequence

Ammo.sequence exposes a simple promise-based API for creating sequential executions for groups of asynchronous methods. It has a very minimalistic design with just two public methods - .chain() and .execute(). Here's an example of ammo.sequence():

```javascript
ammo.sequence()
    .chain(seq => setTimeout(() => {
        console.log('timeout A');
        seq.resolve();
    }, 2500))
    .chain(() => setTimeout(() => {
        console.log('timeout B');
    }, 1500))
    .execute();
```

The code above will first execute the first chained method after 2500 milliseconds and after that it will execute the second chained method after 1500 milliseconds. Without a promise construct wrapping this execution the second method will be executed first. However, ammo.sequence governs exactly for that. So, with it you can create sequential chains of asynchronous functions.

Ammo.sequence has the following options:

```javascript
ammo.sequence()
    .chain(seq => {

        // resolve current method and move on to the next chained method
        seq.resolve(['someData']);

        // reject current method and move on to the next chained method
        seq.reject(new Error('Some error'));
    })
    .chain(seq => {

        // store value, passed by the last successful resolve
        const data = seq.response.value;

        // store error, passed by the last failed reject
        const err = seq.response.error;
    })
    .execute();
```

Using the seq.resolve() and seq.reject() methods you can efficiently control the flow of data through the sequence as well as pass data from one chained method to the other.

`Note:` The last chained method does not need to perform a seq.resolve() or seq.reject() operation. It will be automatically executed as the last chained method. So unless you need to retrieve data from previously chained methods in the sequence, you can skip using the seq argument altogether. This is demonstrated in the first ammo.sequence code example.

## Ammo.select/selectAll

The .select() and .selectAll() methods expose functional, declarative DOM manipulation APIs for easy interactions with the DOM tree. The .select() method is used for single DOM nodes, while .selectAll() is designed to work with DOM node lists. Since both methods expose very similar APIs, we will use .selectAll in most of the examples to follow.

Let's consider the following HTML markup:
```html
<ul class="items">
    <li class="selected active">Item A</li>
    <li class="selected">Item B</li>
    <li>Item C</li>
    <li>Item D</li>
</ul>
```

Now, let's say that we want to get all `li` items, containing class name 'selected'. Here's how to do it with .selectAll():
```javascript

ammo.selectAll('li')
    .filter('selected')
    .each(item => console.log(item));
```

Alternatively, you can filter items using a function:
```javascript

ammo.selectAll('li')
    .filter(item => item.classList.contains('selected'))
    .each(item => console.log(item));
```

`Note:` Both code snippets do exactly the same thing.

In the code above, we selected all `li` items on the page, then we filtered the collection by class name. Only the items, containing class name 'selected' remained in the collection after the filter operation. Finally, we iterated over the filtered collection and outputted the DOM node for each item.

In case we want to get rather than log the results, we need to use the .get() method. Here's how:

```javascript
const selectedItems = ammo.selectAll('li').filter('selected').get();
```

Let's look at another example. This time we want to grab only items that do not have the selected class. We also want to optimize our selectAll query, using a context. Here's how:

```javascript

// define context
const list = ammo.select('.items').get();

// select all li items available in context list
ammo.selectAll('li', list)
    .filter(item => !item.ClassList.contains('selected'))
    .on('click', event => console.log(event.target));
```

This time we utilized the .select() method to get the DOM `ul` list. Then we selected all items in this DOM list by passing the list as the second argument (context) to .selectAll(). After that we filtered and kept only items without class name 'selected'. Finally, we attached an event handler for each of the filtered items. Now, when you click on each of the items without class name 'selected' the callback logging the DOM node for that element will be outputted.

`Note:` Most ammo methods concerned with DOM manipulations expose a second optional argument 'context' (DOM node), which can be used to optimize your DOM queries.

Let's consider another, more advanced, example. For instance, you may want to filter some list items, modify the style and attributes of each item and then iterate all items `over time`. Finally output some text, when the `over time` iteration completes.

```javascript
ammo.selectAll('li', list)
    .filter(item => !item.ClassList.contains('selected'))
    .style('color', 'red')
    .attr('data-iterated', true)
    .async(resolve => setTimeout(() => resolve(), 1000), () => {
        console.log('all items iterated');
    });
```

The above selectAll will filter items with class name 'selected', for each item it will modify its color. Then it will also apply for each item an attribute 'data-iterated' set to true. Finally, each item will be iterated in an asynchronous way over 1000 milliseconds. After the iteration has finished, a message 'all items iterated' will be outputted to the console.

Here's a similar example, achieved with a more functional approach:
```javascript
ammo.selectAll('li', list)
    .filter(item => !item.ClassList.contains('selected'))
    .style('color', (item, index) => index % 2 === 0 ? 'red' : 'green')
    .attr('data-iterated', (item, index) => index % 2 === 0)
    .async(resolve => setTimeout(() => resolve(), 1000), () => {
        console.log('all items iterated');
    });
```

The code does mostly the same thing with the exception of the .style() and .attr() methods. As you can see these methods work with mixed arguments. You can pass as the second argument either a string variable, which will be directly applied over the item's prop. Alternatively, you can use a function (modifier) which contains as arguments some useful data such as the DOM node of the currently iterated item as well as its index. These two arguments allow for more advanced evaluations, which may be required to determine the type of change over the item's style or attribute.

## Notes

Ammo also offers an experimental templating system with the .template() method. The .template() method is especially interesting as it offers utilities for optimized, partially managed DOM updates. However, the usage of ES6 string literals make this method somewhat obsolete.

Ammo comes with many other public methods such as:

- `.recurIter()` - recursive iterator
- `.poll()` - polling iterator
- `.buffer()` - buffer for high-frequency events
- `.eachKey()` - object key iterator
- `.hasKey()` - object key checker
- `.hasMethod()` - object method checker
- `.delegateEvent()` - event delegation method

These methods are heavily used in the code base of Fantasy POS. You are encouraged to explore how they work by examining existing code.
