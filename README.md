# NodeJS

## Imports and Exports

NodeJS allows for the importing and exporting for individual NodeJS files called
modules. In order to import an file we use the require syntax.

```js
const library = require('./library')
```

And in order to export functionality in a module file we must make use of the
module.exports API. Any object or function set as a property of the exports
object is available to be imported in from other files.

```js
const car = {
    brand: 'Ford',
    model: 'Fiesta',
}

module.exports = car
```

We can now import our new car object like so.

```js
const Ford = require('./ford.js')

console.log(Ford.brand)
// -> Ford
```

In example 1 set exports equal to the car object. However we may at times want
to export multiple objects of functions in a file. In order to do this we'd
modify the example above like this.

```js
const car = {
    brand: 'Ford',
    model: 'Fiesta',
}

exports.car = car
```

```js
const items = require('./items')
const car = items.car
```

### Main Module

When a file is run directly through Node.JS `require.main` is set equal to the
directly executed module. Consequently you are able to check if a module was run
directly by checking if `require.main === module`.

### Package Manager Tips

The require module is formatted The require module is formatted in a manner to
support resonable directory structures. It's compatible with package managers
such as `dkpg`, `npm`, `rpm`, etc.. Let's consider an example directory structure
for the foo package.

`/ur/lib/node<some-package>/<some-version>`

### Caching

Modules are cached after the first time they get loaded, this means that every
subsequent call to a module, returns the same object. So multiple calls to a
module does not execute it multiple times. One use of this to to have partial
object that get fileld and passed around through the application. You can do
this for instance to updating a settings configuration in real time.

### Cyclical dependencies

Consider a situation in where we have two modules `a` and `b`. Module a imports
in module b, and module b imports in module a. Let's take a look at how that
scenario plays out using the code snipet below.

**`a.js`:**

```js
console.log('a starting')
exports.done = false
const b = require('./b.js')
console.log('in a, b.done = %j', b.done)
exports.done = true
console.log('a done')
```

**`b.js`:**

```js
console.log('b starting')
exports.done = false
const a = require('./a.js')
console.log('in b, a.done = %j', a.done)
exports.done = true
console.log('b done')
```

**`main.js`:**

```js
console.log('main starting')
const a = require('./a.js')
const b = require('./b.js')
console.log('in main, a.done = %j, b.done = %j', a.done, b.done)
```

**`$ node main.js`:**

```shell
main starting
a starting
b starting
in b, a.done = false
b done
in a, b.done = true
a done
in main, a.done = true, b.done = true
```

When `main.js` is executed it imports in module a, the code in module a begins
execution, but halts because its needs to import in module b. Module b begins
execution, but then starts to import in module a. Now instead of infinitely
looping b recieves an **unfinished** exports object from a. In other words
any properties added to exports by a before it began importing b is what object
b recieves.

### File Modules

Module resolution for files works in the following manner. If the exact filename
if not found Node.js will attempt to load the required filename with the
added extentions `.js`, `.json`, and lastly `.node`.

`.node` files are interpreted as compiled addon modules loaded with
`process.dlopen()`.

A required module prefixed with `'/'` is an absolute path to the file.
`require('/home/<user>/utility/http')`. A module prefixed with `'./'` is a
relative path to a file. And it's relative to the file that is calling `require`.

### Folder Modules

If you wish to create a reusable libraries that you can import into your node
projects, then you need to store the library inside a self contained folder and
define it's entry point.The entry point is what `require()` will use to when
attempting to import it. One way of defining an entry point is through
`package.json`, which should be located at the root of the folder, and contain
a main module. e.g

**`package.json`**

```json
{
    "name": "library name",
    "main": "./app/index.js"
}
```

In no main module is defined inside `package.json` require will then attempt to
locate and load an `index.js` or `index.node` file in the root of the folder. If
it is unable to locate these files, then the import will fail.

### node_modules Module

If the module identifier passed to `require` is not a `core` module, and it does
not begin with `'/'`, `'./'`, or `'../'`, then Node.js starts at the parent
directory of the current module, and adds `/node_modules` to the module path. If
the module is still not located, Node.js will move up once more to a parent
directory and repeat the process until it reaches the root directory and fails.

This is useful because it allows projects to localize dependencies for sub
modules, which can be then imported in independently from the main library.

### Global modules

If the `NODE_PATH` environment is set to a colon delimited list of absolute paths,
then Node.js will look for modules in those locations. Note on `Windows`
`NODE_PATH` is delimited by seimicolons instead of colons.

The following is a list of Global locations also searched for Node.js.

1. `$HOME/.node_modules`
2. `$HOME/.node_libraries`
3. `$PREFIX/lib/node`

The use of global module is mostly discouraged, and you should stick with local
modules.

### Module Wrapper

Before a module's code is executed, Node.js wraps it within a function wrapper
that looks like this.

```js
;(function (exports, require, module, __filename, __dirname) {
    // Module code goes here.
})
```

-   This is to prevent any variables/declarations outside of functions from leaking
    into the global object.

-   Additionally it provides users access to global-looking variables such as
    `module` and `exports`

### Compiled Node Modules

> **TODO Talk about .node files.**

# Where does NPM install packages?

NPM allows users to install packages either `locally` or `globally`. The default
installation type is local, a location installation places the package in the
current source trees `node_modules` folder. Additionally it will add an entry
into the local package.json file. A `global` installation is preformed with the
-g flag.

```bash
npm install -g <package-name>
```

Installing a package in this mannger installs it into the global location. To
find out where this location is on your machine run the `npm root -g` command
