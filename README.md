### react-imported-component ✂

[![CircleCI status](https://img.shields.io/circleci/project/github/theKashey/react-imported-component/master.svg?style=flat-square)](https://circleci.com/gh/theKashey/react-imported-component/tree/master)

### World first any-bundler SSR-friendly loader.
Formerly - simple, but usable Async Component loader to be used with [React-Hot-Loader](https://github.com/gaearon/react-hot-loader).

Easy, universal, and could provide top results without any extra configuration. 
Deliver a better experience with a single import.

[![NPM](https://nodei.co/npm/react-imported-component.png?downloads=true&stars=true)](https://nodei.co/npm/react-imported-component/)

Key features:
 - 🔥 Hot-Module-Replacement friendly.
 - 🌟 Async on client, sync on server.
 - 📦 could handle any bunder, and could load all the used async chunks in one "wave".
 - ✂️ could work with any import statement, passed from anywhere 
 - 🛠 HOC and Component API.
 - 🧙🏻‍♂️ thus, composable.

## Usage

```javascript
import importedComponent from 'react-imported-component';
const Component = importedComponent( () => import('./Component'));
```

The key feature - "could work with any import statement, passed from anywhere". 
All others `full-cream` SSR bundlers relay on `import` statement inside their HOC,
like in the example just above, disallowing any composition.

React-imported-component is different. But still "full-cream".

```javascript
import importedComponent from 'react-imported-component';
const myImportFunction = () => import('./Component')
const Component = importedComponent(myImportFunction);
```

```javascript
import importedComponent from 'react-imported-component';
export default const mySuperImportedFactory = importFunction => importedComponent(importFunction);
//... in another file
mySuperImportedFactory(() => import('./Component'));
```
> Note: due to babel plugin limitation react-imported-component could handle only "() => import('')"
structures, not Promise.all(import('./a'),import('./a')).

To use webpack chunks - just add comments inside an import function
```js
importedComponent( () => import(/* webpackChunkName:'pages' */'./Component'));
```

That is all. Component will be loaded in time and then displayed. And updated on module replacement of course.

## SSR (Server side rendering)
It was usually a headache - async components and SSR, which is currently sync.
React-imported-component break this cycle, making ServerSide rendering sync, and providing
comprehensive ways to rehydrate rendered tree on client. 
It will detect server-side environment and precache all used components.

React-imported component provides two different SSR mode.
#### Magic-less "oneliner"
```js
 // just wait all components to be loaded  
  whenComponentsReady().then(() => {
   ReactDOM.hydrate(<App />,document.getElementById('main'));
  });
```
#### Full-cream SSR-to-Client
 
To enable full cream SSR follow these steps.
1. Add babel plugin 
**On the server**:
```json
{
  "plugins": ["react-imported-component/babel", "babel-plugin-dynamic-import-node"]
}
```

**On the client**:

```json
{
  "plugins": ["react-imported-component/babel"]
}
```
Imported-Component will hook into dynamic imports, providing extra information about files you want to load.

2. Add one more command into package.json
CLI command `imported-components [sources ROOT] [targetFile]`
```js
 "generate-imported-component": "imported-components src src/imported.js"
```
3. Execute this command, and react-imported-component __will generate__ a file with all dynamic imports you have used.
> That's how the magic, bundle independent bundling works.

4. Include this file on client-side, not important for server-side.
```js
import importedComponents from 'src/imported';
```
5. Export "used" components information from server side
```js
  import { printDrainHydrateMarks, drainHydrateMarks } from 'react-imported-component';
  // this action will drain all currently used(by any reason) marks
  // AND print a script tag
  const html = renderToString(<YourApp />) + printDrainHydrateMarks();
  
  // OR return list of usedmarks, and yet again CLEAR the marks list.
  const html = renderToString(<YourApp />) + "<script>const marks="+JSON.stringify(drainHydrateMarks())+"</script>";
```
>! The current version expects you to __synchronously render__ the application, and "drain" used `marks`.
"Drain" will return used marks, and empty the state, making the application ready for the next render.

6. Client side - rehydrate
```js
  import { rehydrateMarks } from 'react-imported-component';

  // this will trigger all marked imports, and await for competition.
  rehydrateMarks().then(() => {
    ReactDOM.render(<App />,document.getElementById('main'));
  });
```

## Comparison
* [react-loadable](https://github.com/thejameskyle/react-loadable).
  * The most popular one  
  * Loader: hybrid (import/require)
  * Front-end: __HRM-not-friendly__.
  * SSR: sync, webpack-bound, sees __all used chunks__.
  * Complex HOC based API. SSR will require additional pass(analyze webpack bundle)

* [react-async-component](https://github.com/ctrlplusb/react-async-component)  
  * The most strange one
  * Loader: import only
  * Front-end: __HRM-not-friendly__
  * SSR: semi-async(async-bootstraper), "waved", sees only currently loaded chunks.
  * Magic HOC based API. All SSR work are hidden behind bootstraper. 

* [loadable-components](https://github.com/smooth-code/loadable-components)
  * The most complex(inside) one
  * Loader: import only
  * Front-end: __HRM-not-friendly__.
  * SSR: semi-async(walkTree), "waved", sees only currently loaded chunks.
  * Simple HOC based API

* [react-universal-component](https://github.com/faceyspacey/react-universal-component)
  * The most "webpack" one. 
  * Loader: hybrid (import/require)
  * Front-end: HRM-friendly.
  * SSR: sync, webpack-bound, synchronous rendering. Sees __all used chunks__.
  * Very complex API
  
* [react-imported-component](https://github.com/theKashey/react-imported-component)
  * This library
  * Loader: import only
  * Front-end: HRM-friendly.
  * SSR: semi-async(preload), bundler-independent, Sees __all used chunks__.
  * Component/HOC API, SSR could require additional pass (extract imports).
  
For now __react-imported-component__ is the most intelligent loader, which will work for any bundler.

## Licence
MIT
