# Fantasy POS - Modules

In Fantasy POS modules are self-contained functionalities, which may or may not be globally available. Modules reside in appDir/modules. Typically, modules manage most of the current view.

## Structure

A module contains several files:

- a javascript file, named after the module - this file contains core module functionality. This file can be found under appDir/modules/{selectedModule}/

- a sass file, named after the module - this file contains the core module styles. This file can be found under appDir/modules/{selectedModule}/

- a module may also have views. Each view is located under appDir/modules/{selectedModule}/ and contains a separate javascript and sass file for its functionality and styles.

In most cases, modules are globally defined objects, accessible from window[{selectedModule}], where `selectedModule` is the name of the module you wish to interact with. Modules' views may be globally accessible or encapsulated within the module.
