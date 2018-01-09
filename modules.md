# Fantasy POS - Modules

In Fantasy POS modules are self-contained, mostly globally available, functionalities. Modules reside in appDir/modules. Typically, modules manage all functionality available at the current route.

## Structure

A module contains several files:

- a javascript file, named after the module - this file contains core module functionality. This file can be found under appDir/modules/{selectedModule}/

- a sass file, named after the module - this file contains the core module styles. This file can be found under appDir/modules/{selectedModule}/

- a javascript file, designed to hold all module templates. This file can be found under appDir/modules/{selectedModule}

- a module may also have views. Each view is located under appDir/modules/{selectedModule}/ and contains a separate javascript and sass file for its functionality and styles.

In most cases, modules are globally defined objects, accessible from window[{selectedModule}], where `selectedModule` is the name of the module you wish to interact with. Modules' views may be globally accessible or encapsulated within the module.

## Architecture

There are two distinct architectures utilized for defining modules in Fantasy POS. Depending on the module's desired functionality, scale and complexity, the first may be more suited than the latter and vise versa.

### Module pattern plus globalization
The simplest way to define a module is by using the module pattern and globalization:

```javascript
// appDir/modules/my-module/my-module.js

// create a global modula
(base => {
    base.myModule = (() => {

        const init = () => {
            console.log(`module 'myModule' initialized.`);
        };

        // Public API
        return { init };
    })();
})(window);
```

The above code creates a globally available module 'myModule' with a public method 'init', which can be invoked from `window.myModule.init()`. This type of architecture is best suited from small modules, due to the fact that the internal structure of the module vastly depends on the programmer's preferences and coding style. You need to define internal structure for separating the model, view and controller. This can be problematic for bigger modules, where strong, predictable structure is vital.

Another weakness of this pattern is that it does not offer a good way of managing module views and templates. Views contain view-specific functionality, separated in a stand alone file. Also, no out-of-the-box facilities for managing state are available with this approach.

 If you need to define views/templates using the above structure, each of the view and the templates wrapper should also be globalized (alternatives are available but they lack good structure). Remember that in Fantasy POS a module often contains several views plus an optional (separate) templates file.

### Ammo.app pattern plus partial globalization
Ammo.app offers a more robust way of creating modules and managing their views:

```javascript
// appDir/modules/my-module/my-module.js

// create global module
(() => {
    const props = {
        name: 'myModule',
        global: true
    };
    const module = ammo.app(props).schema('module');

    module.configure('actions')
        .node('init', () => {
            console.log(`module 'myModule' initialized.`);
        });
})();
```

The above code creates a global ammo.app available under 'myModule'. To call the init() node use:

```javascript
window.myModule.callNode('actions', 'init');
```

Ammo.app offers a superior way of creating modules on a global scale thanks to the exposed rigid API for interacting with the app. With ammo.app, module 'myModule' takes advantage of several key benefits:

- ability to easily manage the module's props and store
- ability to synchronize the module's store with localStorage (via .syncWithPersistentStore() method)
- ability to manage views without globalizing them

Let's look at a more complicated module, which needs 2 views (search, results) and 1 template (index). The module also requires a store and caching for it. We'll be utilizing ammo.app again:

```javascript
// appDir/modules/my-module/my-module.js

// create global module
(() => {
    const props = {
        name: 'myModule',
        global: true,
        storeKey: 'myModuleStore'
    };
    const store = {
        accounts: []
    };
    const module = ammo.app(props, store).schema('module').syncWithPersistentStore();

    module.configure('actions')
        .node('initView', view => {
            const { views } = module.getNodes();

            if (ammo.hasMethod(views, view)) {
                views[view](ammo.app().schema('default'));
            }
        });
})();
```

```javascript
// appDir/modules/my-module/search/index.js

// add view 'search' to module 'myModule'
myModule.addNode('views', 'search', view => {
    view.configure('actions')
        .node('init', () => {
            console.log(`module 'myModule' - view 'search' initialized`);
        });

    view.callNode('actions', 'init');
});
```

```javascript
// appDir/modules/my-module/details/index.js

// add view 'details' to module 'myModule'
myModule.addNode('views', 'details', view => {
    view.configure('actions')
        .node('init', () => {
            console.log(`module 'myModule' - view 'details' initialized`);
        });

    view.callNode('actions', 'init');
});
```

```javascript
// appDir/modules/my-module/my-module-templates.js

// add template 'index' to module 'myModule'
myModule.addNode('templates', 'index', () => {`
    <div data-module="myModule" template"index">Index template</div>
`});
```

First, we create a global ammo.app 'myModule' and we sync its store with localStorage, at key 'myModuleStore'. Now all store data for the module is persistent.  Notice that we also define an .initView() node under 'actions'.

Then, in separate files we define the module's views 'search' and 'details'. Notice than internally, these views are fed their respective view, which is also an ammo.app. The view construct is passed to the selected view in the .initView() method, part of module 'myModule'. Also, note that in this example a view does not have any props or state. This is because, by convention, all state should be defined within the module itself, in this case within 'myModule'. If a view requires access to the module's state, this can be achieved in the following way:

```javascript
// using view 'search'
// appDir/modules/my-module/search/index.js

// add view 'search' to module 'myModule'
myModule.addNode('views', 'search', view => {
    view.configure('actions')
        .node('init', () => {
            console.log(`module 'myModule' - view 'search' initialized`);

            // get module props
            const props = myModule.getProps();

            // get module store
            const store = myModule.getStore();

            // update module store
            myModule.updateStore('accounts', () => []);
        });

    view.callNode('actions', 'init');
});
```

Finally, we create a templates file with a single template 'index'.

In the above example the views and templates are encapsulated within 'myModule'. To invoke a view, you need to use:

```javascript
const initView = window.myModule.getNode('actions', 'initView');
initView('search');
```

To invoke a template, you need to use:

```javascript
const indexTemplate = window.myModule.getNode('templates', 'index');
const html = indexTemplate();
```

`Note:` All views have direct access to the module's store via `myModule.getStore('storeKey')`.

`Note:` All views can write to the module's store view `myModule.updateStore('storeKey', () => [])`.
