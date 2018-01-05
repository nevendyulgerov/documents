# Fantasy POS - Architecture

An overview of the architecture used for the front-end of Fantasy POS.

Fantasy POS is a custom, single page application, utilizing ES6 code to achieve its functionality. It uses a custom router to determine which functionality should be loaded for the current route. Some parts of the Fantasy POS app are utilizing event-driven patterns.

The functionality of Fantasy POS is spread in several directories:

- `./scripts` - contains core functionality
- `./modules` - contains module-related functionality
- `./widgets` - contains widget-related functionality

All scripts residing under these directories are concatenated into a single file - main/main.js. This is the only script file loaded on index.html, the app's entrypoint.

## Scripts

Scripts define core-specific functionality. Under scripts you will find a multitude of javascript files, including:

- api.js
- router.js
- global-events.js

## Modules

Modules define module-related functionality. A module typically contains a sass file, defining its styles and a javascript file, defining its functionality. Oftentimes, a module can also contain a html file, defining the base html markup for the module.

## Widgets

Widgets define widget-related functionality.  A widget typically contains a sass file, defining its styles and a javascript file, defining its functionality. Widgets define their html markup through the use of internal UI functions.
