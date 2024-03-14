---
title: 'Part 2: 100 Front-end Questions and Answers'
date: 2024-02-08T16:22:38+08:00
draft: true
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/0*ge1I6ow4qwBbwv5A"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

## Why is the data property a function instead of an object, and what is the specific reason?

Whether it’s a function or not depends on the scenario. Also, there’s no need to worry about when to write data as a function or object, because vue has internally handled it and output error information in the console.

**Scenario One**: `new Vue({data: ...})`

This scenario is mainly for project entry or when each html page instantiates a Vue, the data here can be in the form of an object or a factory function returning an object. Because the data here only appears once, and there is no problem of data pollution caused by repeated references.

**Scenario Two**: Component scenario options

In the process of generating component vnode, the component will execute the merge strategy during the process of generating the constructor:

```
// Data merge strategy
strats.data = function (
  parentVal,
  childVal,
  vm
) {
  if (!vm) {
    if (childVal && typeof childVal !== 'function') {
      process.env.NODE_ENV !== 'production' && warn(
        'The "data" option should be a function ' +
        'that returns a per-instance value in component ' +
        'definitions.',
        vm
      );
​
      return parentVal
    }
    return mergeDataOrFn(parentVal, childVal)
  }
​
  return mergeDataOrFn(parentVal, childVal, vm)
};
```

If the merging process finds that the data of a subcomponent is not a function, i.e., `typeof childVal !== 'function'` is true, it will output a warning in the console in the development environment and directly return `parentVal`. This suggests that no `data` information from childVal has been merged into `options`.

​We have mentioned before that the data in the component must be a function. Have you ever wondered why?

When we define a component, vue will eventually form a component instance through Vue.extend().

Here we imitate the component constructor, defining the data property in the form of an object.

```
function Component(){
 
}
Component.prototype.data = {
  count : 0
}
```

Create two component instances.

```
const componentA = new Component()
const componentB = new Component()
```

Modify the value of the data property of the componentA, the value in the componentB also changes.

```
console.log(componentB.data.count) // 0
componentA.data.count = 1
console.log(componentB.data.count) // 1
```

The reason for this is that the two share the same memory address. The content modified by componentA also affects componentB.

If we use the form of a function, this situation will not occur (the memory addresses of the objects returned by the function are not the same).

```
function Component(){
  this.data = this.data()
}
Component.prototype.data = function (){
    return {
      count : 0
    }
}
```

Modify the value of the data property of the componentA, the value in the componentB is not affected.

```
console.log(componentB.data.count) // 0
componentA.data.count = 1
console.log(componentB.data.count) // 0
```

vue components may have many instances, using the function to return a new data form, so that the data of each instance object will not be polluted by other instance objects data.

## Do you know about the initialization process of Vue2, what has it done?

When new Vue goes into Vue’s constructor in the `src\core\instance\index.js` file.

```
this._init(options)
```

Then from the prototype method added by Mixin, initMixin(Vue), the one called is the _init prototype method added for Vue

Source code location: `src/core/instance/init.js`

```
function initMixin (Vue) {
  Vue.prototype._init = function (options) {
     var vm = this; // create vm, 
     ...
     // Merge options into vm.$options
     vm.$options = mergeOptions(
       resolveConstructorOptions(vm.constructor), 
       options || {},  
       vm 
     );
 }
 ...
 initLifecycle(vm); // Initialize lifecycle
 initEvents(vm); // Initialize events
 initRender(vm); // Initialize render function
 callHook(vm, 'beforeCreate'); // Execute beforeCreate lifecycle hook
 ...
 initState(vm); // Initialize data, props, methods computed, watch 
 ...
 callHook(vm, 'created'); // Execute created lifecycle hook
   
 if (vm.$options.el) {
      vm.$mount(vm.$options.el); // This is also the key point that will be used later
 }
}
```

**Summary**:

Therefore, from the above functions, the things done by new vue unfold like a flow chart, which are:

- Merge configuration

- Initialize lifecycle

- Initialize events

- Initialize rendering

- Call the beforeCreate hook function

- Init injections and reactivity (at this stage, properties are all injected and bound, and they are transformed into reactivity by $watch, but $el is not yet generated, that is, the DOM is not generated yet)

- Initialize state state (initialize data, props, computed, watcher)

- Call the created hook function.

At the end of initialization, if the el attribute is detected, the vm.$mount method is called to mount vm. The target of the mount is to render the template into the final DOM.

## A rough process of Vue3 initialization

Initial rough process

createApp() => mount() => render() => patch() => processComponent() => mountComponent()

Simplified version of the process

1. Vue.createApp() actually executes renderer’s createApp()

2. renderer is created by the createRenderer method

3. renderer’s createApp() is returned by createAppAPI()

4. After receiving render, createAppApi creates an app instance and defines the mount method

5. mount will call the render function. Convert vnode to real dom

createRenderer() => renderer => renderer.createApp() <= createAppApi()

```
<div id="app"></div>
<script>
    // 3.createAppAPI
    const createAppAPI = render => {
        return function createApp(rootComponent) {
            // return application instance
            const app = {
                mount(rootContainer) {
                    // mount vnode => dom
                    const vnode = {
                        tag: rootComponent
                    }
                    // Execute rendering
                    render(vnode, rootContainer)
                }
            }
            return app;
        }
    }
    // 1. Create createApp
    const Vue = {
        createApp(options) {
            //What actually executed is renderer's createApp()
            // Return app instance
            return renderer.createApp(options)
        }
    }
    // 2. Implement renderer factory function
    const createRenderer = options => {
        // Implement patch
        const patch = (n1, n2, container) => {
            // Get root component configuration
            const rootComponent = n2.tag;
            const ctx = { ...rootComponent.data()}
            // Execute render to get vnode
            const vnode = rootComponent.render.call(ctx);
            // Convert vnode => dom
            const parent = options.querySelector(container)
            const child = options.createElement(vnode.tag)
            if (typeof vnode.children === 'string') {
                child.textContent = vnode.children
            } else {
                //array
            }
            // Append
            options.insert(child, parent)
        }
        // Implement render
        const render = (vnode, container) => {
            patch(container._vnode || null, vnode, container)
            container._vnode = vnode;
        }
        // This object is renderer
        return {
            render,
            createApp: createAppAPI(render)
        }
    }
    const renderer = createRenderer({
        querySelector(el) {
            return document.querySelector(el)
        },
        createElement(tag) {
            return document.createElement(tag)
        },
        insert(child, parent) {
            parent.appendChild(child)
        }
    })
    Vue.createApp({
        data() {
            return {
                bar: 'hello,vue3'
            }
        },
        render() {
            return {
                tag: 'h1',
                children: this.bar
            }
        }
    }).mount('#app')
</script>
```

In Vue3, CreateApp() initializes, mount() is responsible for objectifying, render() transforms vnode into the real dom in the browser, you can understand patch() as a diff algorithm, processComponent() usually processes third-party components in vue, and finally mountComponent() will render our HTML.

## How to write Vue3 responsive API

```
var activeEffect = null;
function effect(fn) {
  activeEffect = fn;
  activeEffect();
  activeEffect = null; 
}
var depsMap = new WeakMap();
function gather(target, key) {
  // void triggering gather when using console.log(obj1.name)
  if (!activeEffect) return;
  let depMap = depsMap.get(target);
  if (!depMap) {
    depsMap.set(target, (depMap = new Map()));
  }
  let dep = depMap.get(key);
  if (!dep) {
    depMap.set(key, (dep = new Set()));
  }
  dep.add(activeEffect)
}
function trigger(target, key) {
  let depMap = depsMap.get(target);
  if (depMap) {
    const dep = depMap.get(key);
    if (dep) {
      dep.forEach((effect) => effect());
    }
  }
}
function reactive(target) {
  const handle = {
    set(target, key, value, receiver) {
      Reflect.set(target, key, value, receiver);
      trigger(receiver, key); // Trigger automatic update when setting values
    },
    get(target, key, receiver) {
      gather(receiver, key); // Collect dependencies when accessing
      return Reflect.get(target, key, receiver);
    },
  };
  return new Proxy(target, handle);
}
​
function ref(name){
    return reactive(
        {
            value: name
        }
    )
}
```

## How do you do SSR rendering in Vue projects

Compared with the traditional SPA (Single-Page Application), the advantages of Server Side Rendering (SSR) mainly include:

- Better SEO, as search engine crawlers can directly view the fully rendered page.

- Faster time-to-content, especially for slow network conditions or slow-running devices.

Vue.js is a framework for building client-side applications. By default, Vue components can be output in the browser to generate DOM and operate DOM. However, the same component can also be rendered as a server-side HTML string, send them directly to the browser, and finally “activate” these static markers into fully interactive applications on the client side.

A server-rendered Vue.js app can also be considered “isomorphic” or “universal” because most of the application’s code can run on both the server and the client.

- Vue SSR is a server-side rendering that has been improved on SPA.

- The page rendered by Vue SSR needs to be activated on the client side to achieve interaction.

- Vue SSR will include two parts: the first screen rendered by the server, and the SPA with interaction.

Using SSR does not have a singleton mode, and a new Vue instance will be created whenever a user makes a request.

Implementing SSR requires implementing server-side first screen rendering and client-side activation.

Server asynchronous data fetching asyncData can be divided into first screen asynchronous fetching and switching component fetching.

The first screen asynchronously fetches data, which should have been completed during server-side pre-rendering.

Switch components fetch data through mixin mixing, and complete data fetching in the beforeMount hook.

## How do you view Vue’s diff algorithm

The diff algorithm is an efficient algorithm for comparing tree nodes at the same level

The overall strategy of diff is: depth first, compare on the same level

Comparisons are only made at the same level, and will not compare across levels

During the comparison process, the loop converges from both ends to the middle

1. When the data changes, the subscriber watcher will call patch to patch the real DOM

2. Call the patchVnode method if they are the same through the isSameVnode judgment

3. PatchVnode does the following operations:

  - Find the corresponding real dom, called el

  - If both have text nodes and are not equal, set the el text node to the text node of Vnode

  - If oldVnode has child nodes and VNode doesn’t, then delete the child nodes of el

  - If oldVnode has no child nodes and VNode has, then add the real child nodes of VNode to el

  - If both have child nodes, execute the updateChildren function to compare the child nodes

4. UpdateChildren mainly does the following operations:

  - Set the head and tail pointers of the new and old VNode

  - Compare the head and tail pointers of the new and old, loop towards the middle, and call patchVnode to repeat the patch process according to the situation, call createElem to create a new node, and find the VNode node with the same key from the hash table and then operate according to the situation.

## What do you need to do to build a Vue project from scratch

- Scaffold: Choose the appropriate initialization scaffold (vue-cli2.0 or vue-cli3.0)

- Request: Data axios request configuration

- Login: Login registration system

- Routing: Route management page

- Data: Vuex global data management

- Permission: Permission management system

- Buried point: Buried point system

- Plugin: The selection and introduction of third-party plugins

- Error: Error page

- Entry: Front-end resources are used directly as static resources, or the server-side template is fetched

- SEO: If SEO is considered, the SSR scheme is recommended

- Components: Base component/Business component

- Style: style preprocessing, extraction of common styles

- Method: Common method extraction

## What are the data types in js and how are the values stored

JavaScript has a total of 8 data types, 7 of which are basic data types: Undefined, Null, Boolean, Number, String, Symbol (new in es6, representing a unique value) and BigInt (new in es10);

There is 1 reference data type — Object (Object is essentially a group of unordered name-value pairs). It contains function, Array, Date, etc. JavaScript does not support any mechanism for creating custom types, and all values will eventually be one of these 8 data types.

Primitive data types: Directly stored in the stack, occupying small space, fixed size, being frequently used data, so they are stored in the stack.

Reference data types: Both stored in the stack and heap, occupying large space, and the size is not fixed. The reference data type stores a pointer in the stack, which points to the starting address of the entity in the heap. When the interpreter is looking for a reference value, it will first search for its address in the stack, and then get the entity from the heap after obtaining the address.

## In JS, Object.prototype.toString.call() is used to determine the data type

```
var a = Object.prototype.toString;
console.log(a.call(2)); // [object Number]
console.log(a.call(true)); // [object Boolean]
console.log(a.call('str')); // [object String]
console.log(a.call([])); // [object Array]
console.log(a.call(function(){})); // [object Function]
console.log(a.call({})); // [object Object]
console.log(a.call(undefined)); // [object Undefined]
console.log(a.call(null)); // [object Null]
```

## What’s the difference between null and undefined?

Firstly, Undefined and Null are both basic data types, each of which has only one value, undefined and null.

undefined represents undetermination, null represents a blank object (not really an object, please see the notes below!). Generally, when a variable is declared but not yet defined, it returns undefined; null is mainly used for initializing variables that may return objects.

In fact, null is not an object, although typeof null will output object, but this is just an long-standing bug in JS. In the earliest version of JS, it used a 32-bit system for performance reasons, used low bits to store variable type information, and 000 at the beginning represented an object. However, null is represented as all-zeros, so it was wrongly judged as an object. Although the code for judging the internal type has changed now, this bug has been passed down.

undefined in js is not a reserved word, which means we can use undefined as a variable name, this practice is very dangerous, it will affect our judgment of the undefined value. However, we can get a safe undefined value through some methods, such as void 0.

When we use typeof to judge the two types, the Null type will return “object”, which is a historical issue. When we use double equals to compare the values of the two types, it returns true, and using triple equals returns false.

## What are the results of valueOf and toString for {} and []?

The valueOf result for {} is {}, and the toString result is [object Object].

The valueOf result for [] is [], and the toString result is “”.

## The Scope and Scope Chain of Javascript

Scope: Scope defines the region where variables are defined. It has a set of rules for accessing variables. These rules manage how the browser engine searches for variables (identifiers) in the current scope and nested scopes.

Scope Chain: The role of the scope chain is to ensure the orderly access to all variables and functions that the execution environment has the right to access. Through the scope chain, we can access the variables and functions of the outer environment.

The essence of the scope chain is a pointer list that points to variable objects. The variable object is an object that contains all the variables and functions in the execution environment. The front end of the scope chain is always the variable object of the current execution context. The variable object of the global execution context (that is, the global object) is always the last object of the scope chain.

When we search for a variable, if it is not found in the current execution environment, we can search backward along the scope chain.

The creation of the scope chain is related to the establishment of the execution context.

## Please discuss your understanding of this, call, apply, and bind

- In the browser, this in the global scope points to the window object;

- In functions, this always points to the object that last called it;

- In constructors, this points to the newly created object through new;

- In call, apply, bind, this is strongly bound to the designated object;

- this in arrow functions is special, the this in the arrow function is the this in the parent scope, not the this at the time of the call. You should know that the first four methods are determined at the time of the call, that is, dynamically, while the this of the arrow function is static and is determined at the time of the declaration;

- apply, call, bind are some built-in APIs inside the functions, which can be used to specify the execution of this for the function. They can also pass parameters.

## What is JavaScript Prototype and Prototype Chain? What are its features?

In js, we use the constructor to create a new object. Each constructor internally has a prototype attribute. This attribute is an object that contains all the properties and methods that can be shared by all instances of this constructor. After we use the constructor to create a new object, the object will internally contain a pointer. This pointer points to the value corresponding to the prototype property of the constructor. In ES5, this pointer is called the prototype of the object. Generally, we should not be able to get this value. However, current browsers have implemented the proto property to allow us to access this attribute. However, we should avoid using this attribute because it is not stipulated in the specification. ES5 added a Object.getPrototypeOf() method. We can use this method to get the prototype of the object.

When we access an attribute of an object and if the attribute does not exist within the object, it will go to its prototype object to find that attribute. This prototype object will also have its own prototype, and it keeps looking this way. This is the concept of the prototype chain. The end of the prototype chain is usually Object.prototype, so this is why newly created objects can use the toString() method and other methods.

Features:

JavaScript objects are passed by reference, and each new object entity we create does not have a copy of its own prototype. When we modify the prototype, the corresponding object will also inherit this change.

## What is a closure, and why should we use it?

A function that can access the internal variables of other functions is called a closure.

A function that can access free variables is called a closure.

**Scenarios**

- As for the use cases of closures, we often use them in daily development.

- Throttling and debouncing functions

- Timer callbacks

**Advantages**

What problem does the closure help us solve?

Internal variables are private, which can isolate scope and prevent data from being polluted

**Disadvantages**

At the same time, closures also bring some downsides.

As its advantage has been discussed, ‘internal variables are private, it can isolate scope’, which means that the garbage collection mechanism cannot clean up internal variables in closures, resulting in memory leaks.

## What are the three event models?

Events are interactive actions that occur when a user operates a web page or some operations of the web page itself. Modern browsers have three event models:

1. **DOM Level 0 model**: This model does not propagate, so there is no concept of event flow, but now some browsers support implementation in a bubbling way. It allows direct definition of listener functions in web pages, or specification of listener functions through js properties. This method is compatible with all browsers.

2. **IE event model**: In this event model, there are two processes for one event, the event handling phase, and the event bubbling phase. The event handling phase will first execute the listening event bound to the target element. Then is the event bubbling phase. Bubbling refers to the event bubbling from the target element to the document while checking whether the nodes passed have bound event listener functions. If there are, then they are executed. This type of model uses attachEvent to add listener functions. It can add multiple listener functions that are executed in the order they were added.

3. **DOM Level 2 event model**: This event model has three processes for one event. The first process is the event capturing phase. Capturing is the event propagating from the document down to the target element, checking if the nodes passed have bound event listener functions. If there are, then they are executed. The next two phases are the same as in the IE event model. In this event model, the function bound to the event is addEventListener. Its third parameter can specify whether the event executes in the capture phase.

## JavaScript arrays and strings have a variety of native methods

**Array methods:**

- push() — Add new items to the end of an array

- pop() — Remove the last item of an array

- shift() — Remove the first item of an array

- unshift() — Add new items to the beginning of an array

- slice() — Returns a new array with a portion of the elements of the original array

- splice() — Add/Remove items from an array

- join() — Join all elements of the array into a string

- reverse() -Reverses the order of the elements in an array

- sort() — Sorts the elements of an array

- map() — Creates a new array with the results of calling a provided function on every element in the array

- filter() — Creates a new array with all elements that pass a test provided by a function

- reduce() — Apply a function against an accumulator and each element in the array (from left to right) to reduce it to a single value.

- forEach() — Executes a provided function once for each array element

**String methods:**

- length() — Returns the length of a string

- charAt() — Returns the character at a specified index in a string

- charCodeAt() — Returns the Unicode of the character at a specified index

- concat() — Concatenate two or more strings

- indexOf() — Returns the position of the first found occurrence of a specified value in a string

- lastIndexOf() — Returns the position of the last found occurrence of a specified value in a string

- match() — Searches a string for a match against a regular expression and returns the matches

- search() — Searches a string for a specified value or regular expression and returns the position of the match

- replace() — Replaces some or all matches with a replacement

- slice() — Extracts a part of a string and returns a new string

- split() — Splits a string into an array of substrings

- substr() — Extracts parts of a string, beginning at the character at the specified position, and returns the specified number of characters

- substring() — Extracts characters from a string between two specified indices

- toLowerCase() — Converts a string to lowercase letters

- toUpperCase() — Converts a string to uppercase letters

## What are the ways to load JavaScript lazily?

The loading, parsing, and execution of JavaScript can block the rendering of the page. Therefore, we hope that JavaScript scripts can be loaded as late as possible to improve the rendering speed of the page.

1. ut JavaScript scripts at the bottom of the document to load and execute them as late as possible.

2. Add the defer attribute to the JavaScript script. This attribute allows the script to load in parallel with the document’s parsing process. After the document has finished parsing, then this script file will be executed. This way, the rendering of the page is not blocked. Multiple scripts set with the defer attribute should be executed in order according to the standard, but in some browsers, it may not be the case.

3. Add the async attribute to the JavaScript script. This attribute allows the script to load asynchronously and will not block the parsing of the page. However, when the script loading is completed, the JavaScript script will be executed immediately. If the document has not finished parsing, it will also block. The execution order of multiple scripts with the async attribute is unpredictable and does not generally follow the order of the code.

4. Dynamic creation of DOM tags. We can listen to the loading events of the document. When the document has finished loading, we can dynamically create script tags to import JavaScript scripts.

## There are four mature module loading schemes in JavaScript now

- The first one is the CommonJS scheme, which uses require to import modules and defines the module’s output interface through module.exports. This module loading solution is a server-side solution, it imports modules in a synchronous way because on the server-side all files are stored on the local disk, so the reading is very fast, so synchronous loading is not a problem. But if it is on the client-side, because the loading of modules uses network requests, using asynchronous loading is more appropriate.

- The second is the AMD scheme, which uses asynchronous loading to load modules, the loading of modules does not affect the execution of subsequent statements, all statements that depend on this module are defined in a callback function, and the callback function is executed after the loading is completed. Require.js implements the AMD specification.

- The third is the CMD scheme, this scheme and the AMD scheme are to solve the problem of asynchronous module loading, sea.js implements the CMD specification. It differs from require.js in the handling of dependencies when defining modules and the timing of executing dependent modules.

- The fourth scheme is proposed by ES6, using import and export to import and export modules.

## The difference between AMD and CMD specifications?
There are two main differences between them.

1. The first aspect is the different handling of dependencies when defining modules. AMD advocates for dependency upfront, which means you need to declare its dependent modules when defining a module. CMD, on the other hand, advocates for just-in-time dependency, meaning that you only require a module when you need it.

2. The second aspect is the differing handling of when dependent modules are executed. Although AMD and CMD both load modules asynchronously, they are different when it comes to the execution timing of modules. Once AMD’s dependent modules are loaded, it starts executing them directly, and the execution order of these modules may not necessarily be the same as how we write them. CMD, on the other hand, does not execute the module immediately after it is loaded, it only downloads it. After all dependent modules have been loaded, it enters the callback function logic. It only executes the corresponding module when it encounters a require statement, so the execution order of the modules is consistent with the order in which we write them.

```
// CMD
define(function(require, exports, module) {
  var a = require("./a");
  a.doSomething();
  ...
  var b = require("./b"); // Dependencies can be written nearby
  b.doSomething();
  // ...
});
// AMD
define(["./a", "./b"], function(a, b) {
  // Dependencies must be written from the beginning
  a.doSomething();
  ...
  b.doSomething();
  // ...
});
```

## ifferences between ES6 modules and CommonJS modules, AMD, and CMD.
1. Syntax

CommonJS uses module.exports = {} to export a module object, require('file_path') to import a module object;
ES6 uses export to export specific data, import to import specific data.

2. CommonJS module outputs a copy of a value, ES6 module outputs a reference to a value

CommonJS module outputs a copy of a value, that is, once a value is output, changes inside the module cannot affect this value.

The operating mechanism of ES6 Modules is different from CommonJS. When the JS engine performs a static analysis of the script, it encounters the module loading command import, it will generate a read-only reference. When the script is actually executed, it will retrieve the value from the loaded module based on this read-only reference. In other words, ES6's import is like Unix system's "symbolic link". If the original value changes, the value loaded by import will also change accordingly. Therefore, ES6 modules are dynamic references and do not cache values. The variables within the module are bound to the module where they are located.

3. CommonJS modules are loaded at runtime, ES6 modules are loaded at compile time

Runtime Loading: CommonJS modules are objects, i.e., modules are loaded in their entirety first to generate an object, and then methods are read from this object. This type of loading is called “runtime loading”.

Compile-time loading: ES6 modules are not objects, they specify the code that is to output explicitly through the export command, and static commands are used when imports. At the time of import, you can specify to load some outputted values instead of loading the whole module. This type of loading is called "compile-time loading".

PS: The CommonJS loads an object (i.e., module.exports property), which is only generated after the script has finished running. ES6 modules, on the other hand, are not objects, their external interfaces are just static definitions that are generated during the static analysis phase of the code.

## JS operation mechanism

Asynchronous task types: macro tasks, micro tasks

Synchronous tasks and asynchronous tasks enter different execution “places”

First, execute the macro tasks in the main thread execution stack

If a micro task is encountered during execution, it enters the Event Table and registers a function, after which it moves into the task queue of micro tasks

After the macro task is executed, all micro tasks in the current micro task queue are immediately executed (in order)

The main thread will continuously retrieve tasks from the task queue, execute tasks, retrieve more, and execute more tasks. This is often referred to as the Event Loop.

## A brief introduction to the garbage collection mechanism of the V8 engine

V8’s garbage collection mechanism is based on generational garbage collection, which is in turn based on the generational hypothesis. This hypothesis has two characteristics, one is that newly created objects easily die young, the other is that objects that do not die will live longer. Based on this hypothesis, the V8 engine divides memory into new generation and old generation.

Newly created objects or objects that have only undergone garbage collection once are referred to as the new generation. Objects that have undergone garbage collection multiple times are referred to as the old generation.

The new generation is divided into ‘From’ and ‘To’ spaces, with ‘To’ typically being idle. When the ‘From’ space is full, the Scavenge algorithm executes garbage collection. When we execute the garbage collection algorithm, the application logic will stop and only continue after the garbage collection ends. This algorithm consists of three steps:

(1) First, check the surviving objects in the ‘From’ space. If the object is alive, check if it meets the criteria for promotion to the old generation. If the conditions are met, it is promoted to the old generation. If the conditions are not met, it is moved to the ‘To’ space.

(2) If the object is not alive, then the space of the object is freed.

(3) Finally, the roles of the ‘From’ space and the ‘To’ space are swapped.

There are two conditions for promoting new generation objects to the old generation:

(1) The first is to check if the object has already undergone a Scavenge recovery. If it has undergone one, the object will be copied from the ‘From’ space to the old generation. If it has not undergone one, it will be copied to the ‘To’ space.

(2) The second is to check if the usage of the ‘To’ space exceeds the limit. When an object is copied from the ‘From’ space to the ‘To’ space, if the ‘To’ space usage is over 25%, the object is directly promoted to the old generation. The reason for setting ‘25%’ is mainly because after the algorithm ends, the two spaces will be swapped. If the memory of the ‘To’ space is too small, it will affect subsequent memory allocation.

The old generation uses the mark-sweep and mark-compact methods. The mark-sweep method first marks the surviving objects in memory and then removes the unmarked objects after the marking ends. Since mark-sweep can cause a lot of memory fragments, which is not conducive to subsequent memory allocation, the mark-compact method was introduced to solve the problem of memory fragmentation.

Since the application logic is paused during garbage collection, the pause time is not too long for the new generation method because of the small memory. However, for the old generation, each garbage collection takes a long time, and the pause will have a significant impact. To solve this problem, V8 introduced an incremental marking method, dividing the pause process into multiple steps. After each small step, it lets the running logic execute for a while, alternating like this.

## What actions can cause a memory leak?

1. Accidental Global Variables

2. Forgotten Timers or Callbacks

3. Out-of-DOM References

4. Closures

## What are the new features in ES6?

- Block Scoping

- Classes

- Arrow Functions

- Template Strings

- Enhanced Object Literals

- Object Destructuring

- Promise

- Modules

- Symbol

- Proxy & Set

- Default Function Parameters

- Spread

## What is an Arrow Function?
What is an Arrow Function?

```
//ES5 Version
var getCurrentDate = function (){
  return new Date();
}
//ES6 Version
const getCurrentDate = () => new Date();
```

Arrow function expressions have a more concise syntax than function expressions and do not have their own this, arguments, super, or new.target. Arrow function expressions are best suited for places where anonymous functions are originally needed, and they cannot be used as constructors.

An arrow function does not have its own this value. It captures the this value of the lexical scope function. If we declare the arrow function in the global scope, the this value is the window object.

## What is a Higher Order Function?

Higher order functions are simply functions that take functions as arguments or return them as results.

```
function higherOrderFunction(param, callback){
    return callback(param);
}
```

## Handwritten call, apply, and bind functions

### Implementing the call Function

Process the edge case:

If the object does not exist, the this points to window.

Assign the “calling function” to the fn property of the object that this points to.

Execute the fn function on the object that this points to, pass in the parameters, and return the results.

```
Function.prototype.mu_call = function (context, ...args) {
    // If the object does not exist, it points to window.
    if (!context || context === null) {
      context = window;
    }

    // Create a unique key value to be used as the internal method name of the context we construct.
    let fn = Symbol();

    // The this points to the function that calls call.
    context[fn] = this;

    // Executing the function and returning the result is equivalent to calling itself as a method of the passed-in context.
    return context[fn](...args);
};

// test
var value = 2;
var obj1 = {
  value: 1,
};
function bar(name, age) {
  var myObj = {
    name: name,
    age: age,
    value: this.value,
  };
  console.log(this.value, myObj);
}
bar.mu_call(null); // {name: undefined, age: undefined, value: 2}
bar.mu_call(obj1, 'tom', '110'); // {name: "tom", age: "110", value: 1}
```

### Implementing the apply Function

Similar to call, the difference lies in the form of parameters.

```
Function.prototype.mu_apply = function (context, args) {
  // If the object does not exist, it points to window.
  if (!context || context === null) {
    context = Window;
  }
  // Create a unique key value to be used as the internal method name of the context we construct.
  let fn = Symbol();
​
  // The this points to the function that calls call.
  context[fn] = this;
​
  // Executing the function and returning the result is equivalent to calling itself as a method of the passed-in context.
  return context[fn](...args);
};
​
// test
var value = 2;
var obj1 = {
  value: 1,
};
function bar(name, age) {
  var myObj = {
    name: name,
    age: age,
    value: this.value,
  };
  console.log(this.value, myObj);
}
bar.mu_apply(obj1, ["tom", "110"]); // {name: "tom", age: "110", value: 1}
```

### Implementing the bind Function

```
Function.prototype.mu_bind = function (context, ...args) {
    if (!context || context === null) {
      context = window;
    }
    // Create a unique key value as the method name inside our constructed context.
    let fn = Symbol();
    context[fn] = this;
    let _this = this;

    const result = function (...innerArgs) {
      // First situation: If the function bound by bind is used as a constructor, 
      // and used through the new operator, do not bind the passed-in this, 
      // but point this to the object that has been instantiated.
      if (this instanceof _this === true) {
        // At this point, this points to the instance of result, 
        // so there is no need to change the pointing of this.
        this[fn] = _this;
        this[fn](...[...args, ...innerArgs]); 
        delete this[fn];
      } else {
        // If it is called as a regular function, then just change this to point to the passed-in context.
        context[fn](...[...args, ...innerArgs]);
        delete context[fn];
      }
    };
    // If what is bound is a constructor, then we need to inherit the prototype's attributes and methods of the constructor, 
    // it can be done using Object.create.
    result.prototype = Object.create(this.prototype);
    return result;
};

function Person(name, age) {
    console.log(name); //'The name passed in by the parameter'
    console.log(age); //'The age passed in by the parameter'
    console.log(this);
}

Person.prototype.say = function () {
    console.log(123);
};

function normalFun(name, age) {
    console.log(name); //'The name passed in by the parameter'
    console.log(age); //'The age passed in by the parameter'
    console.log(this);
    console.log(this.objName); //'The name passed in by obj'
    console.log(this.objAge); //'The age passed in by obj'
}

let obj = {
    objName: 'The name passed in by obj',
    objAge: 'The age passed in by obj',
};

let bindFun = normalFun.mu_bind(obj, 'The name passed in by the parameter');
bindFun('The age passed in by the parameter');
```

## Implementation of Function Currying

```
// Function currying refers to a technique of converting a function that uses multiple parameters into a series of functions that use a single parameter. 

function curry(fn, args) {
  // Get the number of parameters required by the function
  let length = fn.length;

  args = args || [];

  return function() {
    let subArgs = args.slice(0);

    // Concatenate to get all the current arguments
    for (let i = 0; i < arguments.length; i++) {
      subArgs.push(arguments[i]);
    }

    // Determine whether the length of the argument already meets the length of the parameters required by the function
    if (subArgs.length >= length) {
      // If it meets the requirements, execute the function
      return fn.apply(this, subArgs);
    } else {
      // If it does not meet the requirements, recurse to return the curried function and wait for the argument to be passed in
      return curry.call(this, fn, subArgs);
    }
  };
}

// es6 Implementation
function curry(fn, ...args) {
  return fn.length <= args.length ? fn(...args) : curry.bind(null, fn, ...args);
}
```

## Implement a new operator

First, we need to understand what the new operator does:

Firstly, it creates an empty object.

To point the proto of the empty object to the prototype of the constructor.

It makes this point to the newly created object and execute the constructor.

If the execution result has a return value and it is an object, return the execution result, otherwise, return the newly created object.

```
// Code implementation
function my_new(fn,...arg){
    // Firstly, create an empty object
    const obj = {};
    // To point the proto of the empty object to the prototype of the constructor
    Object.setPrototypeOf(obj, fn.prototype)
    // It makes this point to the newly created object and execute the constructor
    const result = fn.apply(obj,arg);
    // If the execution result has a return value and it is an object, return the execution result, otherwise, return the newly created object
    return result instanceof Object ? result : obj;
}

// Validate my_new function
function Dog(name){
    this.name = name;
    this.say = function(){
        console.log('my name is' + this.name);
    }
}

const dog = my_new(Dog, "Shawn");
dog.say() //my name is Shawn
```

## Can you talk about Promise, and can you implement it by handwriting?

Promise is a solution to asynchronous programming, which is more reasonable and powerful than traditional solutions — callback functions and events. It was first proposed and implemented by the community, ES6 wrote it into the language standard, unified its usage, and provided Promise object natively.

Promise, simply put, is a container that holds the result of an event (usually an asynchronous operation) that will only end in the future. From a syntactic point of view, Promise is an object from which you can get messages about asynchronous operations. Promise provides a unified API, and various asynchronous operations can be handled in the same way.

Let’s look at the basic principles of the Promise we are familiar with

- First of all, when we call Promise, it will return a Promise object.

- When constructing a Promise object, you need to pass in an executor function. The main business process of Promise is executed in the executor function.

- If the business running in the executor function is successful, it will call the resolve function; if it fails, it will call the reject function.

- The status of Promise is irreversible. If you call the resolve function and the reject function at the same time, the result of the first call will be adopted by default.

**Considering the Promise/A+ specification, we can also analyze some basic features**

The Promise/A+ specification is quite extensive, here are a few key points for reference. [Promise/A+ specification](https://promisesaplus.com/)

- Promise has three states: pending, fulfilled, and rejected. The default state is pending.

- Promise has a value to save the successful state value and a reason to save the failed state value. It can be undefined/thenable/promise.

- Promises can only go from pending to rejected, or from pending to fulfilled. Once the state is confirmed, it will not change.

- Promise must have a then method, which accepts two parameters, one is the onFulfilled callback for success, and the other is the onRejected callback for failure.

- If an exception is thrown in the then, it will pass this exception as a parameter to the next then’s onRejected callback.

So CustomPromise, can't implement the basic principles of 3, 4. So let's analyze what's missing based on the basic principles and Promise/A+:

- Promise has three states: pending, fulfilled, and rejected.

- The executor calls two methods: reject and resolve

- You also need variables to save the values ​​of success or failure

- Then accepts two parameters, onFulfilled for successful callbacks and onRejected for failed callbacks.

Manually implement promise:

```
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";

class CustomPromise {
  constructor(executor) {
    try {
      executor(this.resolve, this.reject);
    } catch (error) {
      // If there is an error, execute reject directly
      this.reject(error);
    }
  }
  // Why use arrow functions for resolve and reject?
  // If directly called, the this pointer of the regular function points to window or undefined
  // Using arrow function can let this point to the current object instance
  resolve = (value) => {
    // Promise can only go from pending to rejected, or from pending to fulfilled
    if (this.status == PENDING) {
      this.status = FULFILLED;
      this.value = value;
      
      // In resolve, all successful callbacks are taken out and executed
      if (this.onResolvedCallbacks.length) {
        this.onResolvedCallbacks.forEach((fn) => fn());
      }
    }
  };
  reject = (err) => {
    // Promise can only go from pending to rejected, or from pending to fulfilled
    if (this.status == PENDING) {
      this.status = REJECTED;
      this.reason = err;
      // In reject, all failed callbacks are taken out and executed
      if (this.onFulfilledCallbacks.length) {
        this.onFulfilledCallbacks.forEach((fn) => fn());
      }
    }
  };
  // Storing successful callback functions
  onResolvedCallbacks = [];
  // Storing failed callback functions
  onFulfilledCallbacks = [];

  status = PENDING;
  // Value after success
  value = undefined;
  // Value after failure
  reason = undefined;

  then(onFulfilled, onRejected) {
    // If not passed, the default function is used to ensure it is a function type
    onFulfilled =
      typeof onFulfilled === "function" ? onFulfilled : (value) => value;
    onRejected =
      typeof onRejected === "function"
        ? onRejected
        : (reason) => {
            throw reason;
          };

    const thenCustomPromise = new CustomPromise((resolve, reject) => {
      const resolveCustomPromise = (callBack, value) => {
        try {
          const x = callBack(value);
          // If it is equal, it means that it returns itself, throws a type error and returns
          if (resolveCustomPromise === x) {
            return reject(new TypeError("Type Error"));
          }
          // Determine if x is a CustomPromise instance object
          if (x instanceof CustomPromise) {
            // Execute x and call the then method to make its state fulfilled or rejected
            // x.then(value => resolve(value), error => reject(reason))
            // Simplified as
            x.then(resolve, reject);
          } else {
            // Ordinary value
            resolve(x);
          }
        } catch (error) {
          reject(error);
        }
      };
      // Need to judge the status, choose the processing callback function according to the status
      if (this.status == FULFILLED) {
        resolveCustomPromise(onFulfilled, this.value);
      } else if (this.status == REJECTED) {
        resolveCustomPromise(onRejected, this.reason);
      } else if (this.status == PENDING) {
        // When the status is pending, push the then callback into the resolve/reject execution queue and wait for execution
        this.onResolvedCallbacks.push(() =>
          resolveCustomPromise(onFulfilled, this.value)
        );
        this.onFulfilledCallbacks.push(() =>
          resolveCustomPromise(onRejected, this.reason)
        );
      }
    });
    return thenCustomPromise;
  }
  catch(onFulfilled) {
    return this.then(null, onFulfilled);
  }
  finally(callback) {
    return this.then(
      (value) => CustomPromise.resolve(callback()).then(() => value),
      (reason) => CustomPromise.resolve(callback()).then(() => reason)
    );
  }
  //Static resolve method
  static resolve(value) {
    if (value instanceof CustomPromise) return value;
    return new CustomPromise((resolve) => resolve(value));
  }
  //Static reject method
  static reject(reason) {
    return new CustomPromise((resolve, reject) => reject(reason));
  }
  //Static all method
  static all(values) {
    // Used to record the number of successful promises
    let resolveCount = 0,
    // Used to save the results of successful promises
    resolveDataList = [];
    return new CustomPromise((resolve, reject) => {
      function addPromise(key, value) {
        resolveDataList[key] = value;
        resolveCount++;
        if (resolveCount === values.length) {
          resolve(resolveDataList);
        }
      }

      for (let i = 0; i < values.length; i++) {
        let item = values[i];
        if (item instanceof CustomPromise) {
          // The parameter is Promise
          item.then(
            (value) => addPromise(i, value),
            (error) => reject(error)
          );
        } else {
          // The parameter is an ordinary value
          addPromise(i, item);
        }
      }
    });
  }
  //Static race method
  static race(values) {
    return new CustomPromise((resolve, reject) => {
      for (const p of values) {
        p.then(resolve, reject);
      }
    });
  }
  //Static allSettled method
  static allSettled(values) {
    return new Promise((resolve, reject) => {
      let resolveDataList = [],
        resolveCount = 0;
      const addPromise = (status, value, i) => {
        resolveDataList[i] = {
          status,
          value,
        };
        resolveCount++;
        if (resolveCount === values.length) {
          resolve(resolveDataList);
        }
      };
      values.forEach((value, i) => {
        if (value instanceof CustomPromise) {
          value.then(
            (res) => {
              addPromise("fulfilled", res, i);
            },
            (err) => {
              addPromise("rejected", err, i);
            }
          );
        } else {
          addPromise("fulfilled", value, i);
        }
      });
    });
  }
  //Static any method
  static any(values) {
    return new CustomPromise((resolve, reject) => {
      let rejectCount = 0;
      values.forEach((value) => {
        value.then(
          (val) => resolve(val),
          (err) => {
            rejectCount++;
            if (rejectCount === value.length) {
              reject("All promises were rejected");
            }
          }
        );
      });
    });
  }
}
```

## What is async/await and how does it work, can async be handwritten?

Async — Declaration of an asynchronous function

- Automatically converts regular functions to Promise, and the return value is also a Promise object.

- Only after the asynchronous operation inside the async function is completed, then the callback function specified by the then method will be executed.

- You can use await inside asynchronous functions.

Await — Pause the execution of asynchronous functions (var result = await someAsyncCall();)

- Placed before Promise calls, await forces other codes to wait until the Promise is completed and returns a result.

- It can only be used with Promise, not with callbacks.

- It can only be used inside async functions.

## What are the advantages and disadvantages of instanceof, and how is it implemented?

Advantages and disadvantages:

**「Advantages」**: Able to distinguish between Array, Object and Function, suitable for judging custom class instance objects

**「Disadvantages」**: Basic data types such as Number, Boolean, and String cannot be determined

Implementation steps:

- Pass in parameters as instance L on the left and constructor R on the right

- Handle the boundary. If the object to be detected is of a basic type, return false

- Get the prototype of the passed-in parameters separately

- Determine whether the prototype on the left is null, if it is null, return false; if the prototype on both sides is equal, return true, otherwise continue to get the prototype of the prototype on the left.

```
// The parameters passed in are instance L on the left and constructor R on the right
function my_instanceof(L, R) {
  // Handle boundary: check if the instance type is a primitive type
  const baseTypes = ["string", "number", "boolean", "symbol", "undefined"];
​
  if (baseTypes.includes(typeof L) || L === null) return false;
​
  // Get the prototype of the passing parameters separately
  let Lp = L.__proto__;
  let Rp = R.prototype; // Only functions have the prototype property
​
  // Judge the prototype
  while (true) {
    if (Lp === null) return false;
    if (Lp === Rp) return true;
    Lp = Lp.__proto__;
  }
}

// validate
const isArray = mu_instanceof([],Array);
console.log(isArray); //true
const isDate = mu_instanceof('2023-01-09',Date);
console.log(isDate); // false
```

## Throttling and Debounce in JavaScript

### Debounce

Function debounce is to execute the callback after the event is triggered for n seconds. If it is ‘triggered within n seconds’, then ‘retime’.

```
function debounce(fn, wait) {
  let timeout;
  return function () {
    clearTimeout(timeout);
    timeout = setTimeout(() => {
      fn.apply(this, arguments);
    }, wait);
  };
}

// Test
function handle() {
  console.log(Math.random());
}
// When the window size changes, trigger the debounce function and execute the handle function
window.addEventListener('resize', debounce(handle, 1000));
```

### Throttle

When an event is triggered, it ensures that the function is only called once within a certain time period. For example, when scrolling the page, send a request every once in a while.

Implementation steps:

- Pass in parameters for execute function fn and wait time wait.

- Save initial time now.

- Return a function, if it exceeds the waiting time, execute the function and update now to the current time.

```
function throttle(fn, wait, ...args) {
  var pre = Date.now();
  return function () {
    // The function may have parameters
    var context = this;
    var now = Date.now();
    if (now - pre >= wait) {
      // Point the 'this' of the execution function to the current scope
      fn.apply(context, args);
      pre = Date.now();
    }
  };
}
// Test
var name = 'mu';
function handle(val) {
  console.log(val + this.name);
}
// Scroll the mouse, trigger the throttle function and execute the handle function
window.addEventListener('scroll', throttle(handle, 1000, 'mu'));
```
