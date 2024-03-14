---
title: '28 Vue Interview Questions with Detailed Explanations'
date: 2024-01-22T16:22:38+08:00
draft: true
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/0*j1QzZDqnrYjXm2hG"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: "This article is written from the perspective of a front-end interviewer, summarizing some important features and principles of the Vue framework ..."
---

This article is written from the perspective of a front-end interviewer, summarizing some important features and principles of the Vue framework in the form of questions. The intention is to help both the author and the reader self-assess the extent of their understanding of Vue. This article is organized in order of difficulty, and it’s recommended for readers to follow the chapters sequentially, but experts can navigate as they please. After reading this article, it is hoped that readers will have some thought-provoking insights, a better understanding of their proficiency in Vue, and be able to make up for any knowledge gaps to gain a better grasp of Vue.

## Can you explain your understanding of SPA single page applications and their pros and cons?

SPA (single-page application) only loads the corresponding HTML, JavaScript, and CSS when the web page is initialized. Once the page is loaded, SPA will not reload or jump due to user operations. Instead, it uses the routing mechanism to change the HTML content and the UI’s interactions with users, avoiding page reloading.

**Advantages:**

- Excellent and quick user experience. The content changes do not require reloading the entire page, avoiding unnecessary redirection and repeated rendering;

- Based on the above, SPA puts less pressure on the server;

- Clear separation of front-end and back-end responsibilities, with the front-end handling interactive logic, and back-end dealing with data processing.

**Disadvantages:**

- Initial load time is long: To implement single-page web application features and display effects, JavaScript, CSS need to be loaded simultaneously when loading the page, and part of the page is loaded on demand;

- Forward and backward route management: Since the single-page application displays all content on one page, the browser’s forward and backward functions cannot be used. All page switching requires its stack management;

- Larger SEO difficulty: Since all content is dynamically replaced and displayed on one page, it has a natural disadvantage in terms of SEO.

## What is the difference between v-show and v-if?
v-if is true conditional rendering, as it ensures that event listeners and child components within the conditional block are properly destroyed and rebuilt during the switch. It is also lazy: if the condition is false at initial rendering, it does nothing. It only starts rendering the conditional block when the condition first becomes true.

v-show is much simpler. No matter what the initial condition is, the element will always be rendered, and it simply switches based on the CSS “display” property.

So, v-if is suitable for scenarios where the condition rarely changes at runtime and where frequent switching is not needed. v-show, on the other hand, is suitable for situations that require very frequent condition switching.

## How are Class and Style dynamically bound?

Class can be dynamically bound using object syntax and array syntax:

Object

```
<div v-bind:class="{ active: isActive, 'text-danger': hasError }"></div>
```
```
data: {
  isActive: true,
  hasError: false
}
```

Array

```
<div v-bind:class="[isActive ? activeClass : '', errorClass]"></div>
```
```
data: {
  activeClass: 'active',
  errorClass: 'text-danger'
}
```

Style can also be dynamically bound using object syntax and array syntax:

Object

```
<div v-bind:style="{ color: activeColor, fontSize: fontSize + 'px' }"></div>
```
```
data: {
  activeColor: 'red',
  fontSize: 30
}
```

Array

```
<div v-bind:style="[styleColor, styleSize]"></div>
```
```
data: {
  styleColor: {
     color: 'red'
   },
  styleSize:{
     fontSize:'23px'
  }
}
```

## How to understand the one-way data flow in Vue?

All props form a one-way downstream binding between parent and child props: updates to the parent prop will flow downstream to the child components, but not the other way around. This prevents the state of the parent component from being accidentally changed from the child component, making your application’s data flow difficult to understand.

In addition, every time the parent component is updated, all props in the child component will be refreshed to the latest values. This means that you should not change props within a child component. If you do, Vue will give a warning in the browser console. When the child component wants to modify, it can only distribute a custom event through $emit, and the parent component modifies it after receiving it.

There are two common scenarios for trying to change a prop:

- This prop is used to pass an initial value; the child component then wants to use it as a local prop data. In this case, it’s best to define a local data attribute and use this prop as its initial value:

```
props: ['initialCounter'],
data: function () {
  return {
    counter: this.initialCounter
  }
}
```

- This prop is passed in with a raw value that needs to be converted. In this case, it’s best to use the value of this prop to define a computed property.

```
props: ['size'],
computed: {
  normalizedSize: function () {
    return this.size.trim().toLowerCase()
  }
}
```

## What’s the difference between computed and watch, and what scenarios are they used in?

`Computed`: It’s a computed property that depends on other property values, and the value of computed has a cache. Only when the property values it depends on change will the value of computed be recalculated the next time it’s obtained;

`Watch`: It serves more as an “observer”. It’s akin to a callback for monitoring certain data. The callback is executed for follow-up operations each time the monitored data changes.

Usage scenarios:

- When we need to perform numerical calculations and depend on other data, we should use computed, because we can take advantage of the cache characteristics of computed to avoid recalculating every time the value is obtained;

- When we need to perform asynchronous or costly operations when data changes, we should use watch. Using the watch option allows us to perform asynchronous operations (like accessing an API), limit the frequency at which we perform this operation, and set intermediary states before we get the final result. These are things that computed properties can’t do.

## If you directly assign a value to an item in an array, can Vue detect the change?

Due to JavaScript’s limitations, Vue cannot detect the changes of an array as follows:

- When you directly set an array item via the index. For example: vm.items[indexOfItem] = newValue

- When you modify the length of the array. For example: vm.items.length = newLength

To solve the first issue, Vue provides the following methods:

```
// Vue.set
Vue.set(vm.items, indexOfItem, newValue)
// vm.$set，Vue.set的一个别名
vm.$set(vm.items, indexOfItem, newValue)
// Array.prototype.splice
vm.items.splice(indexOfItem, 1, newValue)
```

To solve the second issue, Vue provides the following methods:

```
// Array.prototype.splice
vm.items.splice(newLength)
```

## Could you discuss your understanding of the Vue lifecycle?

a. What is lifecycle?

A Vue instance has a complete lifecycle, which includes a series of processes from its creation, data initialization, template compilation, DOM mounting to rendering, updating, rendering, and unmounting. We call this the lifecycle of Vue.

b. The function of each lifecycle stage.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*GuPF9Iep5TpjcfNEfxcouA.png)

## What is the execution order of lifecycle hook functions in Vue’s parent components and child components?
The execution order of lifecycle hook functions in Vue’s parent and child components can be categorized into four parts:

- Load Rendering Process

    Parent beforeCreate -> Parent created -> Parent beforeMount -> Child beforeCreate -> Child created -> Child beforeMount -> Child mounted -> Parent mounted

- Subcomponent Update Process

    Parent beforeUpdate -> Child beforeUpdate -> Child updated -> Parent updated

- arent Component Update Process

    Parent beforeUpdate -> Parent updated

- Destruction Process

    Parent beforeDestroy -> Child beforeDestroy -> Child destroyed -> Parent destroyed

## In which lifecycle should the asynchronous request be called?

You can call it in the created, beforeMount, and mounted hook functions, because in these three hook functions, the data has been created and you can assign the data returned by the server. However, I recommend calling asynchronous requests in the created hook function for the following reasons:

- You can get the server data faster, reducing the page loading time.

- SSR does not support the beforeMount and mounted hook functions, so placing them in created helps with consistency.

## At what stage can we start to access and manipulate the DOM?

Before the mounted hook function is called, Vue has already mounted the compiled template to the page, so you can access and manipulate the DOM in mounted. For a more detailed diagram of Vue’s specific lifecycle, you can refer to the following. Once you understand the actions at each stage of the entire lifecycle, questions related to the lifecycle won’t be a challenge in interviews.

## Can a parent component listen to the lifecycle of a child component?

For example, if you have a parent component Parent and a child component Child, and if the parent component listens for the child component’s mounted event to perform some logic, you can implement it with the following method:

```
// Parent.vue
<Child @mounted="doSomething"/>
    
// Child.vue
mounted() {
  this.$emit("mounted");
}
```

The above requires manually triggering the parent component’s event with $emit. An even simpler way is to listen to the @hook when the parent component references the child component, as shown below:

```
//  Parent.vue
<Child @hook:mounted="doSomething" ></Child>
doSomething() {
   console.log('The parent component listens to the mounted hook function ...');
},
    
//  Child.vue
mounted(){
   console.log('The child component triggers the mounted hook function. ...');
},    
    
// The output order above is:
// Child component triggers the mounted hook function ...
// The parent component listens to the mounted hook function ...
```

Of course, the @hook method is not only able to listen to the mounted event. Other lifecycle events, such as: `created`, `updated`, etc. can also be monitored.

## What do you know about keep-alive?

Keep-alive is a built-in component of Vue, which can retain the state of the components it wraps, preventing them from re-rendering. It has the following features:

- It is usually used with routing and dynamic components to cache components.

- It provides `include` and `exclude` attributes, both of which support strings or regular expressions. `include` indicates that only components with matching names will be cached, `exclude` means any components with matching names will not be cached. The `exclude` attribute has higher priority than ‘include’.

- It corresponds to two hook functions: `activated` and `deactivated`. When the component is activated, the `activated` hook function is triggered. When the component is removed, the deac`tivated hook function is triggered.

## Why is data a function in a component?

Why must the data in the component be a function that returns an object, while in a new Vue instance, data can directly be an object?

```
// data
data() {
  return {
 message: "Child",
 childName:this.name
  }
}

// new Vue
new Vue({
  el: '#app',
  router,
  template: '<App/>',
  components: {App}
})
```

This is because components are meant to be reused, and in JavaScript, objects have reference relationships. If data in a component is an object, then the scopes are not isolated and data property values in the child components can affect each other. If the data option in a component is a function, then each instance can maintain an independent copy of the returned object, and the data property values between component instances will not interfere with each other. As for the data in a new Vue instance, it is not going to be reused, so there is no problem with referring to objects.

## What is the principle behind v-model?

We mainly use the `v-model` directive in vue projects to create two-way data binding on form input, textarea, select elements, etc. It is known that `v-model` is essentially a syntactic sugar, using different properties and emitting different events for different `input` elements internally:

- Text and textarea elements use the value property and input event;

- Checkbox and radio use the checked property and change event;

- Select fields use value as a prop and change as an event.

Taking the input form element as an example:

```
<input v-model='something'>
    
equal to
<input v-bind:value="something" v-on:input="something = $event.target.value">
```

If it’s in a custom component, v-model by default will use a prop called `value` and an event called `input`, as shown below:

```
// Parent：
<ModelChild v-model="message"></ModelChild>


// Child：
<div>{{value}}</div>
props:{
    value: String
},
methods: {
  test1(){
     this.$emit('input', 'jack')
  },
},
```

## What are the ways of communication between Vue components?

Communication between Vue components is one of the common topics tested in interviews. This question is a kind of open-ended one, the more methods you answer, the more points you earn, indicating that you are more proficient in Vue. Communication between Vue components refers to the following three categories: parent-child component communication, cross-generational component communication, and sibling component communication. Below we will introduce each communication method and indicate which type of component communication is applicable.

(1) `props` / `$emit` is applicable to parent-child component communication

This is a basic method for Vue components. I believe most students are familiar with it, so I won’t go into detail here.

(2) ref and `$parent` / `$children` for parent-child component communication

- ref: If used on common DOM elements, the reference points to the DOM element; if used on child components, the reference points to the component instance

- `$parent` / `$children` Accesstoparent/childinstances

(3) EventBus(`emit` / `$on`) is applicable for parent-child, cross-generational, and sibling component communication

This method uses an empty Vue instance as the central event bus (event center) to trigger and listen to events, thereby implementing communication between any components, including parent-child, cross-generational, and sibling components.

(4) `attrs`/`listeners` is suitable for cross-generational component communication

- `$attrs`: Contains attribute bindings (excluding class and style) in the parent scopes that are not recognized (and fetched) by prop. When a component does not declare any props, `$attrs` will contain all bindings (excluding class and style) from the parent scope, and can be passed to the inside components via `v-bind=”$attrs”`. Usually, it is used in combination with the inheritAttrs option.

- `$listeners`: It includes the v-on event listeners from the parent scope (without the .native modifier). It can be passed to the internal component through `v-on=”$listeners”`.

(5) `provide` / `inject` is suitable for cross-generational component communication

The ancestor component provides variables through provider, and then the descendant component injects variables through inject. The `provide` / `inject` API mainly solves the communication problem between cross-level components, but its usage scenarios are mainly for child components to obtain the state of the upper-level components, and establish a kind of relationship between active provision and dependency injection in components across levels.

(6) Vuex is suitable for parent-child, cross-generational, and sibling component communication

Vuex is a state management pattern developed specifically for Vue.js applications. The core of each Vuex application is the store. The `store` is basically a container that contains most of the state in your application.

The state storage of Vuex is reactive. When Vue components read the state from the store, if the state in the store changes, the corresponding components will also be efficiently updated accordingly.
The only way to change the state in the store is to explicitly commit mutation. This makes it easy for us to track changes in each state.

## Have you used Vuex before?

Vuex is a state management pattern developed specifically for Vue.js applications. The core of each Vuex application is the store. The `store` is essentially a container, containing most of the state (state) in your application.

(1) Vuex’s state storage is reactive. When Vue components read the state from the store, if the state in the store changes, the corresponding components will also be efficiently updated.

(2) The only way to change the state in the store is to explicitly commit a mutation. This allows us to easily track every state change.

The main modules include:

- State: Defines the data structure of the application state, and the default initial state can be set here.

- Getter: Allows components to get data from the Store. The mapGetters helper function simply maps the getters in the store to local computed properties.

- Mutation: The only way to change the state in the store, and it must be a synchronous function.

- Action: Used to commit mutations, instead of changing the state directly, it can contain any asynchronous operations.

- Module: Allows the single Store to be split into multiple stores and saved in a single state tree at the same time.

## Have you used Vue SSR before? What is SSR?

Vue.js is a framework for building client-side applications. By default, Vue components can be output in the browser, generating and operating DOM. However, the same component can also be rendered as server-side HTML strings, sent directly to the browser, and finally “activate” these static markups into fully interactive applications on the client side.

That is: SSR roughly means that the work of rendering the tags into a whole HTML fragment on the client side is completed on the server side. The HTML fragment formed by the server directly returns to the client side, which is called server-side rendering.

The advantages and disadvantages of server-side rendering SSR are as follows:

(1) Advantages of server-side rendering:

- **Better SEO**: Because the content of the SPA page is obtained through Ajax, and the search engine crawler does not wait for the completion of the Ajax asynchronous request before crawling the page content, so in the SPA, the content obtained from the page through Ajax cannot be crawled; while SSR is returned directly by the server The rendered page (data is already included in the page), so the search engine crawler can crawl the rendered page;

- **Faster content arrival time (faster first screen loading)**: SPA will wait until all js files compiled by Vue have been downloaded before it starts rendering the page. File downloading needs a certain amount of time, so the first screen rendering needs a certain amount of time; SSR is rendered directly by the server Direct return display of the good page, no need to wait for downloading js files and rendering, etc., so SSR has a faster content arrival time;

(2) The disadvantages of server-side rendering:

- **More development condition restrictions**: For example, server-side rendering only supports two hook functions of beforCreate and created, which will cause some external expansion libraries to need special treatment before they can run in server-side rendering applications; and it can be deployed on any static file servers With the completely static single-page application SPA, the server-side rendering application needs to be in a Node.js server running environment;

- **More server load**: Rendering a complete application in Node.js will obviously occupy more CPU resources (CPU-intensive) than a server that only provides static files. Therefore, if you expect to use it in a high-traffic environment, please prepare for the corresponding server load. And wisely adopt caching strategies.

## How many routing modes are there in vue-router?

vue-router has 3 routing modes: hash, history, abstract, as shown in the source code below:

```
switch (mode) {
  case 'history':
 this.history = new HTML5History(this, options.base)
 break
  case 'hash':
 this.history = new HashHistory(this, options.base, this.fallback)
 break
  case 'abstract':
 this.history = new AbstractHistory(this, options.base)
 break
  default:
 if (process.env.NODE_ENV !== 'production') {
   assert(false, `invalid mode: ${mode}`)
 }
}
```

The explanations of the 3 routing modes are as follows:

- hash: Use URL hash value for routing. Support all browsers, including browsers that do not support the HTML5 History API;

- history: Depends on HTML5 History API and server configuration. For more details, you can check the HTML5 History mode;

- abstract: Supports all JavaScript runtime environments, such as Node.js server-side. If no browser API is found, the router will automatically be forced into this mode.

## Can you talk about the implementation principle of the commonly used hash and history routing modes in vue-router?

(1) Hash mode implementation principle
The implementation of early web front-end routing is based on location.hash. The principle is simple, the value of location.hash is the content after # in the URL. For example, in the following website, its value of location.hash is `#search`:

```
https://www.word.com#search
```

The implementation of the hash routing mode is mainly based on the following features:

- The hash value in the URL is just a state of the client, that is, when sending a request to the server, the hash part will not be sent;

- Changes in the hash value will add a record in the browser’s access history. Therefore, we can control the switch of the hash through the browser’s back and forward buttons;

- You can use the a tag and set the href attribute, when the user clicks this tag, the hash value of the URL will change; or use JavaScript to assign a value to loaction.hash, changing the hash value of the URL;

- We can use the hashchange event to listen for changes in the hash value and navigate (render) the page accordingly.

(2) History mode implementation principle

HTML5 provides the History API to implement URL changes. Among them, the two most important APIs are history.pushState() and history.repalceState(). These two APIs can operate the browser’s history record without refreshing. The only difference is that the former adds a history record, and the latter directly replaces the current history record, as shown below:

```
window.history.pushState(null, null, path);
window.history.replaceState(null, null, path);
```

The implementation of the history routing mode is mainly based on the following features:

- Use the two APIs, pushState and repalceState, to implement URL changes;

- We can use the popstate event to listen for changes in the url, and then jump (render) the page;

- history.pushState() or history.replaceState() will not trigger the popstate event, at this time we need to manually trigger the page jump (render).

## What is MVVM?
Model–View–ViewModel (MVVM) is a software architectural design pattern developed by Microsoft WPF and Silverlight architects Ken Cooper and Ted Peters. It is a method of simplifying User Interface Programming in an Event-Driven way. It was first released by John Gossman, also one of the architects of WPF and Silverlight, in 2005, via his blog.

MVVM originates from the classic Model–View–Controller (MVC) pattern. The appearance of MVVM has promoted the separation of front-end development and back-end business logic, greatly improving the efficiency of front-end development. The core of MVVM is the ViewModel layer. It acts like a transfer station (value converter), which is responsible for converting the data objects in the Model to make the data easier to manage and use. This layer is bidirectionally data bound to the view layer and interacts with the Model layer through the interface request. Such as shown in the diagram below:

![](https://miro.medium.com/v2/resize:fit:1000/format:webp/1*Pb7oyFJQq4Mw681gXBVM1Q.png)

(1) View Layer

The View refers to the User Interface (UI). The frontend is primarily constructed using HTML and CSS.

(2) Model Layer

The Model refers to the data model, loosely referring to various business logic processing and data operations conducted in the backend. For the front-end, it’s the API interface provided by the backend.

(3) ViewModel Layer

The ViewModel is the view data layer organized, generated, and maintained by front-end developers. In this layer, front-end developers perform transformation processing on the Model data obtained from the backend, perform secondary encapsulation, to generate a view data model that meets the expected use of the View layer. It should be noted that the data model encapsulated by the ViewModel includes both the state and behavior of the view, while the data model in the Model layer only includes the state. For example, what a certain part of the page displays; what happens when the page is loaded; what happens when this part is clicked; what happens when this part scrolls, etc., all belong to the view behavior (interaction). Both the view state and behavior are encapsulated in the ViewModel. Such encapsulation allows the ViewModel to describe the View layer completely.

MVVM framework has implemented two-way binding so that the content of the ViewModel is showcased in the View layer in real time. Front-end developers no longer have to inefficiently and tediously manipulate the DOM to update the view. The MVVM framework has taken care of the dirtiest and most tiring part. We developers only need to handle and maintain the ViewModel. Updating the data will automatically update the view accordingly. Therefore, what the View layer shows is not the data from the Model layer, but the data of the ViewModel. The ViewModel is responsible for interacting with the Model layer, which completely decouples the View layer and the Model layer. Such decoupling is essential for success and is a crucial part of the front-end and back-end separation plan implementation.

Here, we use a Vue instance to demonstrate the specific implementation of MVVM. Those with Vue development experience should intuitively understand:

(1) View Layer

```
<div id="app">
    <p>{{message}}</p>
    <button v-on:click="showMessage()">Click me</button>
</div>
```

(2) ViewModel Layer

```
var app = new Vue({
    el: '#app',
    data: {   
        message: 'Hello Vue!', 
    },
    methods: {  
        showMessage(){
            let vm = this;
            alert(vm.message);
        }
    },
    created(){
        let vm = this;
        ajax({
            url: '/your/server/data/api',
            success(res){
                vm.message = res;
            }
        });
    }
})
```

(3) Model Layer

```
{
    "url": "/your/server/data/api",
    "res": {
        "success": true,
        "name": "IoveC",
        "domain": "www.cnblogs.com"
    }
}
```

## How does Vue implement data two-way binding?

The two-way binding of data in Vue mainly refers to: when data changes, the view is updated; when the view changes, data is updated, as shown in the following figure:
That is:

![](https://miro.medium.com/v2/resize:fit:688/format:webp/1*u9PanmMYoW8_5TgYrmk0Zg.png)

When the content of the input box changes, the data in the Data changes synchronously. That is, the change from View => Data.

When data in the Data changes, the content of the text node changes synchronously. That is, the change from Data => View.

Among them, when the View changes and updates the Data, it can be implemented through event listening, so the main task of Vue’s data two-way binding is how to update the View based on Data changes.

Vue mainly uses the following four steps to achieve data two-way binding:

- **Implement a listener Observer**: Traverse the data object, including the properties of the child attribute object, and use Object.defineProperty() to add a setter and getter to each property. In this way, assigning a value to a property of this object will trigger the setter, so you can listen to the data changes.

- **Implement a parser Compile**: Parse the Vue template instructions, replace all variables in the template with data, then initialize the rendering of the page view, bind the updating function to each instruction corresponding node, add subscribers who listen to data, once the data changes, receive notification, call the update function to update the data.

- **Implement a subscriber Watcher**: Watcher subscriber is a bridge for communication between Observer and Compile, the main task is to subscribe to the message of property value changes in Observer, when receiving the message of property value changes, trigger the corresponding update function in the parser Compile.

- **Implement a subscriber Dep**: The subscriber adopts the publish-subscribe design pattern to collect subscriber Watcher and manage the listener Observer and subscriber Watcher uniformly.

The flowchart for the above four steps is shown below.

![](https://miro.medium.com/v2/resize:fit:1340/format:webp/1*eSzrxi2Z1c-psTivK1McCQ.png)

## How does Vue framework implement monitoring of objects and arrays?

If asked how Vue implements data two-way binding, everyone will definitely answer that by using Object.defineProperty() to intercept data. However, Object.defineProperty() can only intercept properties, cannot hijack the entire object, similarly, it can’t intercept arrays. But as we know when using the Vue framework, Vue can detect changes in objects and arrays (operations of some methods), so how does it achieve this? Let’s take a look at the relevant code below:

```
/**
   * Observe a list of Array items.
   */
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])  // The "observe" function is used to monitor changes in data.
    }
  }
/**
   * Recursively traversing the properties.
   */
  let childOb = !shallow && observe(val)
```

By looking at the above part of the Vue source code, we now know that the Vue framework achieves the monitoring of objects and arrays (operations of some methods) by using Object.defineProperty() through traversing arrays and recursively traversing objects.

## Comparisons between Proxy and Object.defineProperty are as follows:

Advantages of Proxy include:

- Proxy can directly listen to objects rather than attributes;

- Proxy can directly listen to array changes;

- Proxy has as many as 13 interception methods, not limited to apply, ownKeys, deleteProperty, has and so on, which are not available in Object.defineProperty;

- Proxy returns a new object, we can only operate the new object to achieve the purpose, while Object.defineProperty can only traverse object properties to modify directly;

- As a new standard, Proxy will receive continuous performance optimization from browser manufacturers, also known as the performance dividend of the new standard.

Advantages of Object.defineProperty include:

Good compatibility, supports IE9, while Proxy has browser compatibility problems and cannot be smoothed by polyfill. That’s why the author of Vue declared that it needs to wait for the next major version (3.0) to rewrite with Proxy.

## How does Vue use vm.$set () to solve the problem that new properties of objects cannot be responsive?

Due to the limitations of modern JavaScript, Vue cannot detect the addition or deletion of object properties. Since Vue will convert properties to getter/setter during the initialization of instances, properties must exist in the data object for Vue to convert them into responsive. However, Vue provides Vue.set (object, propertyName, value) / vm.$set (object, propertyName, value) to add responsive properties to objects, so how does the framework itself implement it?

We look at the corresponding Vue source code: vue/src/core/instance/index.js

```
export function set (target: Array<any> | Object, key: any, val: any): any {
  // target is array  
  if (Array.isArray(target) && isValidArrayIndex(key)) {
    // Modify the length of the array to avoid incorrect execution of splice() due to index > array length
    target.length = Math.max(target.length, key)
    // Use the splice mutation method of the array to trigger responsiveness  
    target.splice(key, 1, val)
    return val
  }
  // If key already exists, modify the property value directly  
  if (key in target && !(key in Object.prototype)) {
    target[key] = val
    return val
  }
  const ob = (target: any).__ob__
  // If target itself is not responsive data, assign value directly
  if (!ob) {
    target[key] = val
    return val
  }
  // Handle the property with responsiveness
  defineReactive(ob.value, key, val)
  ob.dep.notify()
  return val
}
```

By reading the above source code, we know that the principle of vm.$set is:

- If the target is an array, the splice method of the array is directly used to trigger responsiveness;

- If the target is an object, we will first determine whether a property exists and whether the object is responsive. Ultimately, if responsive processing is needed for the property, defineReactive method is called for responsive processing (defineReactive method is the method called by Vue when initializing the object and adding getter and setter to the object property using Object.defineProperty).

## Advantages and disadvantages of virtual DOM

Advantages:

- Guarantees a lower performance limit: The framework’s virtual DOM needs to adapt to any operations that the upper-level API might generate. Its implementation of some DOM operations must be universal, so its performance is not the best. However, it is much better than crude DOM operations, so the framework’s virtual DOM can at least guarantee that when you don’t need to optimize manually, it can still provide decent performance, i.e., guarantee the lower limit of performance;

- No need to manually manipulate the DOM: We no longer need to manually operate the DOM. We only need to write good View-Model code logic. The framework will help us update the view in a predictable way based on the virtual DOM and data bi-directional binding, greatly improving our development efficiency;

- Cross-platform compatibility: The virtual DOM is essentially a JavaScript object, while the DOM is strongly related to the platform. In comparison, the virtual DOM can be more conveniently operated cross-platform, such as server rendering, weex development, etc.

Disadvantages:

- Cannot perform extreme optimization: Although the virtual DOM + reasonable optimization is sufficient to meet the performance requirements of most applications, in some highly performance-demanding applications, the virtual DOM cannot achieve targeted extreme optimization.

## The implementation principle of Virtual DOM

The implementation principle of Virtual DOM mainly includes the following three parts:

- Simulating the real DOM tree with JavaScript objects: This presents an abstraction of the real DOM.

- The Diff algorithm: This is used to compare the differences between two virtual DOM trees.

- The Patch algorithm: This is used to apply the differences between the two virtual DOM objects to the real DOM tree.

## What does key do in Vue?

In Vue, the `key` serves as a unique identifier for `vnodes`. Through this `key`, our ‘diff’ operations can become more accurate and faster. The diff process in Vue can be summarized as follows: ‘oldCh’ and ‘newCh’ each have two variables — `oldStartIndex`, `oldEndIndex` and `newStartIndex`, ‘newEndIndex’. New nodes and old nodes will be compared pairwise, that is, there are four comparison methods: `newStartIndex` and `oldStartIndex`, newEndIndex and `oldEndIndex`, `newStartIndex` and `oldEndIndex`, newEndIndex and `oldStartIndex`. If all the four comparisons do not match, and if `key` is set, a further comparison will be made based on `key`. In the comparison process, the traversal is toward the middle. Once StartIdx > EndIdx, it indicates that either ‘oldCh’ or ‘newCh’ has been traversed, and the comparison ends.

Therefore, the role of `key` in Vue is: `key` serves as a unique identifier for `vnode` in Vue. Through this `key`, our `diff` operations can become more accurate and faster.

Accurate: Since with `key`, in-place reuse is not applied. In the comparison of `a.key === b.key` in `sameNode` function, in-place reuse can be avoided. Therefore, it will be more accurate.

Faster: Using the uniqueness of `key` to generate a `map` object to get the corresponding node is faster than the traversal method, as shown in the source code.

## What optimizations have you performed on the Vue project?

(1). Code level optimization

- Distinguish between v-if and v-show usage scenarios

- Distinguish between computed and watch usage scenarios

- A key must be added to the item for v-for iteration, and the simultaneous use of v-if should be avoided

- Long list performance optimization

- Event destruction

- Image resource lazy loading

- Route lazy loading

- Import third-party plugins as needed

- Optimize the performance of infinite lists

- Server-Side Rendering (SSR) or Pre-rendering

(2). Webpack level optimization

- Compression of images by Webpack

- Reduce redundant code from ES6 to ES5

- Extract common code

- Template precompilation

- Extract the CSS of the component

- Optimize SourceMap

- Analysis of the building results

- Vue project compilation optimization

(3). Basic Web technology optimization

- Enable gzip compression

- Browser caching

- Use of CDN

- Use Chrome Performance to find performance bottlenecks
