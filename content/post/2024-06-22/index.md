---
title: 'Vue3 Performance Optimization — Code-Level Optimization'
date: 2024-06-22T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*KZsES-HkGUvu6TkmfmNMBA.png"
categories: ["Code","Javascript"]
tags: ["Code","Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "Through its data two-way binding and virtual DOM technologies, the Vue framework takes care of the most tedious and dirtiest part of front-end development ..."
---

## Distinguishing Use Cases between v-if and v-show

`v-if` is true conditional rendering because it ensures that event listeners and child components within the conditional block are properly destroyed and recreated during the toggling process, it is also lazy: if the condition is false during the initial render, it does nothing — only when the condition turns true for the first time does it start rendering the conditional block.

`v-show` is much simpler; regardless of the initial condition, the element is always rendered and simply toggles based on the CSS display property.

Therefore, `v-if` is suitable for scenarios where the condition rarely changes at runtime, and there is no need to frequently toggle the condition; `v-show` is suitable for scenarios that require very frequent condition toggling.

However, in actual development, we are unable to determine whether a scenario (Dialog) is frequently toggled.

Therefore, in practice, deciding between `v-if` and `v-show` should be based on their characteristics. `v-if` should be used when a component needs to be created at a specific time and destroyed at a specific time. Conversely, `v-show` can be used when a component only needs to be created once.

## Difference in Use Cases Between Computed and Watch

- `computed`: `computed` properties depend on other property values, and the value of `computed` is cached. The `computed` value is only recalculated the next time it is accessed if any of its dependent properties have changed.

- `watch`: Serves more as an observer, similar to a callback for data changes. The callback is executed for subsequent operations whenever the watched data changes.

Use Cases:

When we need to perform calculations that depend on other data, we should use computed. This takes advantage of the caching feature of computed properties, avoiding redundant recalculations every time the value is accessed.

When we need to perform asynchronous or resource-intensive operations in response to data changes, we should use watch. The watch option allows us to perform asynchronous operations (accessing an API), limit the frequency of these operations, and set intermediate states before we get the final result. These are capabilities that computed properties do not have.

## When using v-for in iterations, a key must be added to each item, and avoid using v-if at the same time

(1). A `key` must be added to each item in a `v-for` loop

When iterating over list data for rendering, each item should have a unique key to help Vue.js’s internal mechanism accurately locate the item in the list. This facilitates a quicker diff process when the state updates by comparing the new state values with the old ones.

(2). Avoid using `v-if `together with `v-for`

`v-for` has a higher priority than v-if. Iterating over the entire array every time can affect performance, especially when only a small part of it needs to be rendered. In such cases, it should be replaced with computed properties where necessary.

## Use v-memo Directive

The v-memo directive allows for caching the results of computed properties or component methods to avoid unnecessary recalculations. The method or computed property value will only be recalculated and the interface updated when the specified attribute values change.

You can refer to this article for more information — [Understanding v-memo in Vue3](https://medium.com/stackademic/understanding-v-memo-in-vue3-a91ea445a9cf)

## Long List Performance Optimization

Lazy loading and virtual lists serve as two common optimization strategies for long lists, each having its own limitations and applicable scenarios:

Lazy loading has a basic concept similar to pagination (but provides a better user experience on mobile, hence its broader application), i.e., not rendering all the data in the list at once but appending new batches of data to the end of the list as the user scrolls to its bottom. However, it cannot reflect the total length of the list, which is both a limitation and an advantage, as it can encourage users to continuously seek new content. In contrast, although virtual lists are virtual, the scrollbar length is real, accurately reflecting the total length of the list.

While lazy loading avoids the performance downfall associated with rendering a large number of DOM elements at once, it does not fully address the performance issues related to handling a large number of DOM elements. As users persistently access new content, the page will still host a large number of DOM elements. However, virtual lists ensure that the number of DOM elements on the page remains roughly constant, thereby solving this issue. Virtual lists mean that aside from the real rendered list items in the viewport, the others outside of visibility are “virtual” and not rendered, greatly reducing the number of DOM elements on the page and naturally enhancing rendering performance.

Since lazy loading only adds new DOM elements without altering existing ones, and virtual lists often modify (even destroy and recreate) DOM elements, lazy loading therefore has better performance when list items need to load static resources or perform time-consuming operations.

Moreover, it’s also possible to implement lazy loading within virtual lists.

You can use [vue-virtual-scroller](https://github.com/Akryum/vue-virtual-scroller) to implement your own virtual list.

## Event Destruction

When a Vue component is destroyed, it automatically cleans up its connections with other instances, unbinds all its directives and event listeners, but this is limited to the component’s own events. If events are added using addEventListener in JavaScript, they will not be automatically destroyed. We need to manually remove these event listeners when the component is destroyed to avoid memory leaks, such as:

```
created() {
  addEventListener('click', this.click, false)
},
beforeDestroy() {
  removeEventListener('click', this.click, false)
}
```

Events registered within hooks can also be uniformly destroyed using the effectScope API. You can refer to this article for more information — [Understanding effectScope in Vue3](https://medium.com/@kxming/understanding-effectscope-in-vue3-f22836608cd4)

## Lazy Loading of Image Resources

For pages with a large number of images, in order to speed up the page loading time, often we need to avoid loading images that are not in the visible area of the page until they scroll into view. This significantly improves page loading performance and enhances the user experience. In our projects, we use the Vue plugin vue-lazyload:

### Install

```
yarn add vue3-lazyload
```

### Usage

`main.js`:

```
import { createApp } from 'vue'
import VueLazyLoad from 'vue3-lazyload'
import App from './App.vue'

const app = createApp(App)
app.use(VueLazyLoad, {
  // options...
})
app.mount('#app')
```

`component-a.vue`:

```
<template>
  <img v-lazy="your image url" />
</template>
<!-- or -->
<template>
  <img v-lazy="{ src: 'your image url', loading: 'your loading image url', error: 'your error image url' }">
</template>
```

For more usage methods and configurations, you can check the official documentation — [vue3-lazyload](https://github.com/murongg/vue3-lazyload).

## Router Lazy Loading

Because Vue packs all route components into one file during compilation, it results in the need to download a large file upon initial loading. By using router lazy loading, route components can be split into multiple smaller files, which are only downloaded as needed, thereby reducing the initial resource load. Loading component resources on demand improves resource utilization and enhances user experience.

The implementation of router lazy loading:

```
import Vue from 'vue'
import Router from 'vue-router'

Vue.use(Router)
const routes = [
  {
    path: '/home',
    name: 'Home',
    // Lazy loading the Home component
    component: () => import('@/components/Home.vue')
  },
  {
    path: '/about',
    name: 'About',
    // Lazy loading the About component
    component: () => import('@/components/About.vue')
  }
]

export default new Router({
  mode: 'history',
  base: process.env.BASE_URL,
  routes
})
```

In this example, the `import()` syntax is used to load the component only when the route is visited, which effectively implements lazy loading for the route components.

## AsyncComponent

Due to Vue loading child components first before loading the parent component, if the loading time for a child component is too long, it can lead to a prolonged period of blankness on the page. The use of async components can solve this issue.

Vue 3 provides `defineAsyncComponent`, which allows for a more concise definition of asynchronous components. Just pass a parameter that returns the `import()` function, which dynamically loads the component when needed. This approach reduces the need to manually write code for asynchronous loading.

You can refer to this article for more information — [Understanding Async Components in Vue3](https://medium.com/@kxming/understanding-async-components-in-vue3-ca2fb5e5fe16)

## Server-Side Rendering (SSR) or Pre-rendering

Server-side rendering refers to the process where Vue performs the rendering of the entire HTML fragment on the server side, rather than on the client side, and the server-generated HTML fragment is directly returned to the client. This process is known as server-side rendering (SSR).

(1) Advantages of server-side rendering:

- Better SEO: Since the content of SPA pages is fetched through Ajax, and search engine crawlers do not wait for Ajax to asynchronously complete before crawling page content, the content fetched by Ajax in SPAs cannot be crawled. On the other hand, SSR returns a page that has already been rendered on the server (with data already included in the page), making it possible for search engine crawlers to fetch the rendered page.

- Faster time to content (quicker first screen load): In SPAs, the rendering of the page only begins after all the Vue-compiled js files have been downloaded, which requires some time, hence there’s a delay in first screen rendering. SSR, being rendered directly by the server and returned for display, does not require waiting for the js files to be downloaded and then rendered, thus achieving a faster content delivery time.

(2) Disadvantages of server-side rendering:

- More development constraints: For example, server-side rendering only supports the beforeCreate and created lifecycle hooks, which necessitate special handling of some external libraries to make them work in server-rendered applications. Moreover, unlike fully static SPAs that can be deployed on any static file server, SSR applications need to run in a Node.js server environment.

- Increased server load: Rendering a full application in Node.js obviously occupies more CPU resources compared to just serving static files, thus if you anticipate high traffic, be prepared for the increased server load and wisely adopt caching strategies.
