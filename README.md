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

If no main module is defined inside `package.json` require will then attempt to
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

## Where does NPM install packages?

NPM allows users to install packages either `locally` or `globally`. The default
installation type is local, a location installation places the package in the
current source trees `node_modules` folder. Additionally it will add an entry
into the local package.json file. A `global` installation is preformed with the
-g flag.

```bash
npm install -g <package-name>
```

Installing a package in this mannger installs it into the global location. To
find out where this location is on your machine run the `npm root -g` command.

## Package.json guide

The only established rule for what can be inside a package.json file is that its
contents must be valid json. Otherwise its contents vary based on the tools and
packages required by a project. Common properties listed in most package.json
files are listed blow.

-   `version` indicates the current version of a package.
-   `name` sets the application/package name
-   `main` sets the entry point for the application
-   `private` when `true` prevents the app/package from accidentally being published
-   `scripts` defines a set of node scripts you can run.
-   `dependencies` sets a list of npm packages installed as dependencies
-   `devDependencies` sets a list of npm packages installed as development dependencies
-   `engines` sets which version of Node.js this package/app works on
-   `browserslist` is used to tell which browsers and their versions are supported

### Package versions

Npm package versions are denoted by 3 period delimited numbers. The first number
is referred to as the major release number, the second version is the minor release,
and lastly the third version is the minor release version.

### Semantic Versioning using npm

All Node.js packages use semantic versioning which means a version number is
composed of 3 digits `x.y.z`.

-   x: the first digit is the major version
-   y: the second digit is the minor version
-   z: the third digit is the patch version

Each semantic version follows a rule.

-   an increase in the major number means you are pushing out an incompatible
    api change.
-   an increase in the minor version means you are adding a fuctionality in a
    backwards compatible manner
-   an increase in the patch version means you are making a backwards compatible
    bug fix.

Its important to adhere to these rules, because the package.json file uses that
format to determine what versions are installable, based on the symbol that is
prefixed infront of the version number.

-   `^` versions prefixed by this symbol mean that any version that has the same
    major number can be installed. e.g `^13.0.0` means you can install `13.0.1`
    `13.1.1` but not `14.0.0` or above.
-   `~` This prefix means that you can install any patch version but the major and
    minor versions are frozen. So `~13.0.0` means you can install `13.0.1` but
    `13.1.0` is not valid.
-   `>` This prefix means that you can install any patch version that is higher
    than the one you specified.
-   `>=` This prefix means that you can install any patch version that is higher
    than or equal to the patch version you specified.
-   `<=` This prefix means that you can install any patch version that is lower
    than or equal to the patch version you specified.
-   `<` This prefix means that you can install any patch version that is lower
    than the patch version specified.
-   `=` This prefix means you accept only that exact version specified.
-   `-` This prefix means you accept a range of versions. e.g `2.1.0 - 2.6.2`
-   `||` This prefix can be used to combine rules. e.g `< 2.1 || > 2.6`.

## The npx Node.js Package Runner

`npx` lets you run code built with Node.Js and published through the npm
registry.

### Easily run local commands

Node.js developers used to publish global packages in order to use commands
easily through their terminals. The problem with this approach is that you were
then pidgeon holed into using a specific version of that command. With npx you
can locally install comands and npx will search through the `node_modules`
folder and excute the desired command, without needed to knw any paths.

Another beneifit of `npx` is that is allows you to run commands, without having
to install them. Running `npx commandname` will automatically run the specified
code by fetching it from the npm registry and executing the code.

This is useful because.

1. You don't have to install any commands.
2. You can run different versions of the same command, using the @version syntax.

```bash
npx node@10 -v #v10.18.1
```

### Run arbitrary code snippets directly from a URL.

`npx` does not limit you to the packages on the npm registry.

You can run code that sits on a GitHub gist for example.

```bash
npx https://gist.github.com/zkat/4bc19503fe9e9309e2bfaa2c58074d32
```

The command at that url is composed of 3 files. Simple optional `Readme.md` file
that details/specs out the command. An `index.js` file that contains the code to
execute. And a `package.json` that specifies the `bin` property so npx knows
what file to run.

**`index.js`**

```js
#!/usr/bin/env node
console.log('yay gist')
```

**`package.json`**

```json
{ "name": "npx-is-cool", "version": "0.0.0", "bin": "./index.js" }
```

Note how the index.js file begins with `#!/usr/bin/env node`, this is necessary
without this the scripts are started without the node executable!

## The Node.js Event Loop

The Node.js JavaScript code runs on a single thread. So only one thing happens
at any given time. Despite this Node.js still provides the ability to run things
asynchronously and have non-block I/O. It does this through it's event-loop
which offloads operations to the system kernel whenever possible.

Additionally each browser tab generally has it's own event loop, this is to
prevent a single tab from blocking your entire browser.

**`Event Loop Flow`**

```
   ┌───────────────────────────┐
┌─>│           timers          │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │     pending callbacks     │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
│  │       idle, prepare       │
│  └─────────────┬─────────────┘      ┌───────────────┐
│  ┌─────────────┴─────────────┐      │   incoming:   │
│  │           poll            │<─────┤  connections, │
│  └─────────────┬─────────────┘      │   data, etc.  │
│  ┌─────────────┴─────────────┐      └───────────────┘
│  │           check           │
│  └─────────────┬─────────────┘
│  ┌─────────────┴─────────────┐
└──┤      close callbacks      │
   └───────────────────────────┘
```

Each phase of the Even loop is depicted above in boxes, and each phase has a
FIFO queue of callbacks to execute. The event loop will execute callbacks that
belong to its current phase until the queue has been exhausted or the maximum
number of callbacks has executed, at which point it will move onto the next
phase.

### Phase Overview

-   **timers**: this phase executes callbacks scheduled by `setTimeout()` and
    `setInterval()`.
-   **pending callbacks**: executes I/O callbacks deferred to the next loop
    iteration.
-   **idle, prepare**: used internally only.
-   **poll**: retrieve new I/O events; execute I/O related callbacks.
-   **checks**: `setImediate()` callbacks are invoked here.
-   **close callbacks**: some close callbacks, e.g socket.on('close')`.

### Phases in Detail

#### timers

A timer specifies the **threshold** after which a provided callback may be
executed and not the exact time it will run. Timer callbacks will run as early
as they can be scheduled after the specified amount of time has passed; however
Operating Systems scheduling or the running of other callbacks may delay them.
The poll phase controls when timers are executed. Lets consider the following
example to better understand that.

Lets say you schedule a timeout to execute after a 100ms threshold, then your
script starts asynchronously reading a file which takes 95ms:

**`scheduler`**

```js
const fs = require('fs')

function someAsyncOperation(callback) {
    // Assume this takes 95ms to complete.
    fs.readFile('/path/to/file', callback)
}

const scheduledTimeoutAt = Date.now()
setTimeout((
    console.log(`I was scheduled ${Date.now() - scheduledTimeoutAt}ms ago`)
) => {}, 100)

someAsyncOperation (() => {
    const start = Date.now()

    while (Date.now() - start < 10) {
        // kill 10ms
    }
})
```

When the `poll` phase begins, its queue is empty (`fs.readFile()`
has not completed), so it will wait for the number of ms remaining until the
soonest timer's threshold is reached. While it is waiting, 95ms pass,
`fs.readFile()` finishes reading the file and its callback which takes 10ms to
complete is added to the `poll` queue and executed. When the callback finishes,
there are no more callbacks in the queue, so the event loop will see that the
threshold of the soonest timer has been reached then wrap back to the `timers`
phase to execute the timer's callback. In this example, you will see that the
total delay between the timer being scheduled and its callback being executed
will be 105ms.

To prevent the `poll`phase from starving the event loop, libuv (the C library
that implements the Node.js event loop and all of the asynchronous behaviors
of the platform) also has a hard maximum (system dependent) before it stops
polling for more events. This means that if the readSync call took longer than
100ms to complete, the event loop would execute the timer callback and then
check back on if readSync has completed to launch its callback.

#### poll

The poll phase has two main functions:

1. Calcuate how long it should block and poll for I/O, then
2. Processing events in the `poll` queue.

When the event loop enters the `poll` phase and there are no timers scheduled,
one of two things will happen.

-   If the `poll` queue is **not empty**, the event loop will iterate through its
    queue of callbacks executing them synchronously until either the queue has
    been exhausted, or the system-dependent hard limit is reached.

-   If scripts **have not** been scheduled by `setImmediate()`, the event loop
    will wait for callbacks to be added to the queue, then execute them
    immediately.`

Once the `poll` queue is empty the event loop will check for timers whose time
threshold have been reached. If completed timers exist, the event loop will
wrap back to the timers phase and execute all timer callbacks.

#### check

This phase allows a person to execute callbacks immediately after the `poll`
phase has completed. If the `poll` phase is idle and scrupts have been queued
with `setImmediate()`, the event loop may continue to the `check` phase rather
than eaiting.

`setImmediate()` is actually a special timer that runs in a seperate phase of
the event loop. It uses a libuv API that schedules callbacks to excute after the
pill phase has completed.

Generally, as the code is executed, the event loop will eventually hit the `poll`
phase where it will wait for an incoming connection, request, etc. However, if
a callback has been scheduled with `setImmediate()` and the `poll` phase becomes
idle, it will end and continue to the `check` phase rather than waiting for
`poll` events.

#### close callbacks

If a socket or handle is closed abruptly (e.g `socket.destroy()`), the `close`
event will be emitted in this phase. Otherwise it will be emitted via
`process.nextTick()`

### setImmediate() vs setTimeout()

`setImmediate()` and `setTimeout()` are similar, but behave in different ways
depending on when they are called.

-   `setImmediate()` is designed to execute a script once the current `poll`
    phase completes.

-   `setTimeout()` schedules a script to be run after a minimum threshold in ms
    has elapsed.

The order in which the timer are executed will vary depending on the context
in which they are called. If both are called from within the main module, then
timing will be bound by the performance of the process (which can be impacted
by other applications runnong on the machine).

However when scheduled in the context of an I/O Cycle, `setImmediate` will always
execute first.

**`SetImmediate in I/O Cycle`**

```js
const fs = require('fs')

fs.readFile(__filename, () => {
    setTimeout(() => {
        console.log('timeout')
    }, 0)
    setImmediate(() => {
        console.log('immediate')
    })
})
```

```bash
$ node timeout_vs_immediate.js
immediate
timeout

$ node timeout_vs_immediate.js
immediate
timeout
```

### process.nextTick()

`process.nextTick()` is not part of the event loop, since it is not shown on the
diagram above. Instead there is a `nextTick` queue which is processed after the
current operation is completed, regardless of the current phase of the event
loop. An operation is defined as a transition from the underlying C/C++ handler,
and handling the JavaScript that needs to be executed.

That means any time you call `process.nextTick()` in a phase, all callbacks
passed to `process.nextTick()` will be resolved before the event loop continues.
This can be bad cause it allows you to "starve" your **I/O** by making recursive
`process.nextTick()` calls, which would prevent the loop from reaching the poll
phase.

Here is a real world example of using Process.nextTick()

```JS
const server = net.createServer(() => {}).listen(8080);

server.on('listening', () => {});
```

When passing the port to listen the Port is bound immediately.So the listening
callback could be called immediately.The problem with this is that the callback has
not been set at that time.To get around this the listen callback is queued using
Process.netTick.

#### process.nextTick() vs setImmediate()

We've gone over two scheduling functions that are similar in nature.

-   `process.nextTick()` fires immediately on the same phase
-   `setImmediate()` fires on the following iteration or 'tick' of the event loop.

The name of these two functions should be swapped, but this is an artifact of
the past that cannot be changed. Node.JS doc recommend using `setImmediate`
in all cases because it's easier to reason about.

#### Why use process.next()?

There can be use main reasons:

1. Allow users to handle errors, cleanup any unneeded resources, or perhaps
   try the request again before the event loop continues.
2. Sometimes its necessary to allow the a callback to run after the call stack
   has been unwound but before the next event loop.

### Blocking the event loop

Any JavaScript code that takes too long to return back control to the event loop
will block the execution of all JavaScript code on the page, even block the UI
thread, making it so users cannot click around, scroll the page, etc..

Almost all the I/O primitives in JavaScript are non-blocking. Network request,
filesystem operations, and so on. Being blocking is the exception, and this is
why JavaScript is based so much on callbacks, and more recently promises and
async/await.

### The call stack

The call stack is LIFO, (Last In, First Out). The event loop continuously checks
the `call stack` to see if there is any function that needs to run.

While doing so, it adds any function call it finds to the stack and executes
each in order.

### Queuing function execution (setTime())

You are able to defer a function call until the call stack is clear. This is
useful to avoid blocking the CPU on intensive task and let other functions be
executed while performing a havy calculation.

**`index.js`**

```js
const bar = () => console.log('bar')

const baz = () => console.log('baz')

const foo = () => {
    console.log('foo')
    setTimeout(bar, 0)
    baz()
}

foo()

// Output
//-> foo
//-> baz
//-> bar
```

![Call Stack With Defer](/figures/deferStack.png?raw=true 'Defer Call Stack')

If the function you pass to `setTimeout` takes in paramters you can pass those
as additional arguments.

**`setTimeout() Example`**

```js
const myFunction = (firstParam, secondParam) => {
    // do something
}

setTimeout(myFunction, 2000, firstParam, secondParam)
```

setTimeout returns an id, which can be used to cancel the scheduled function.

**`Canceling setTimeout`**

```js
const id = setTimeout() => {
    //run after 2 seconds
}, 2000)
```

### The Message Queue

When setTimeout is called, the browser or Node.js starts a timer, when the timer
expires the callback function is placed into the `Message Queue`. The Message
queue is where user-initiated events like clicks, key, scroll, etc.. are
queued.Even DOM events like onload are queued into the message queue.

The Event loop gives priority to the call stack, and only after the call stack
begins empty does it begin to queue up items from the Message Queue into the
call stack.Set timeout is non-blocking, if you pass it a time of 3 seconds,
the execution of your code is not halted for the 3 second period. Instead the
wait happens seperately inside the browser / node.js environment.

### ES6 Job Queue

ECMAScript 2015 introduced the concept of the Job Queue, which is used by
Promises (also introduced in ES6/ES2015). It's a way to execute the result of an
async function as soon as possible, rather than being put at the end of the call
stack.

Promises that resolve before the current function ends will be executed right
after the current function.

**`index.js`**

```js
const bar = () => console.log('bar')

const baz = () => console.log('baz')

const foo = () => {
    console.log('foo')
    setTimeout(bar, 0)
    new Promise((resolve, reject) =>
        resolve('should be right after baz, before bar')
    ).then((resolve) => console.log(resolve))
    baz()
}

foo()

//-> Output
//-> foo
//-> baz
//-> should be right after baz, before bar
//-> bar
```

### Understanding process.nextTick()

Each time the event loop takes a full trip, we call that a tick. Consequently
when we pass a function to `process.nextTick()`, we instruct the engine to
invoke the function at the end of the current operation, before the next event
loop tick starts.

Use `nextTick()` when you want to make sure that in the next event loop
iteration that code is already executed.

### Understanding setImmediate()

When you want to execute a piece of code asynchronously, but as soon as possible
one option is to use the `setImmediate()` function provided by Node.js.

```js
setImmediate(() => {
    // run something
})
```

Any function passed to `setImmediate` is executed in the next iteration of the
event loop. This means that it will be executed after `process.nextTick()`
but either before or after `setImmediate(() => console.log('hi'), 0)`, depending
on various factors.

### Scheduling Reoccuring Task (setInterval)

`setInterval` works exactly in the same manner as `setTimeout` with the
exception that the provided callback runs indefinately at an interval which is
passed as the second argument.

**`setInterval`**

```js
setInterval(() => {
    // do something every 2 seconds.
}, 2000)
```

Like `setTimeout`, `setInterval` also returns an id which can be used to
terminate the scheduled callback.

**`terminate interval`**

```js
const id = setInterval(() => {
    // do something every 2 seconds
}, 2000)

clearInterval(id)
```

Consider the scenario where setInterval is passed a callback which runs for
the same exact duration every time. This is the ideal scenario and results in
a timegraph similar to the one pictured below.

![Ok-SetInterval](/figures/setinterval-ok.png?raw=true 'Perfect Scheduling')

Now lets say the duration of the function execution varies significantly, this
may be the case when it comes to networking request. In such scenarios we'd
have a timegraph similar to the one pictured below.

![Ok-SetInterval](/figures/setinterval-overlapping.png?raw=true 'Perfect Scheduling')

In order to avoid this, you can schedule a recursive setTimeout to be called
when the back function finishes.

**`recursive setTimeout`**

```js
const muFunction = () => {
    setTimeout(myFunction, 1000)
}

setTimeout(myFunction, 1000)
```
