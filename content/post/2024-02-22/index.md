---
title: 'Understanding Async Components in Vue3'
date: 2024-02-22T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*SoXuFiQrG7Mex4MN6hmO3g.jpeg"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: "In Vue3, there are many functions related to responsiveness, such as toRef, toRefs, isRef, unref, etc."
---

## Background

When developing Vue projects, most people will use components. In the parent component, the child components are generally loaded in sequence, where the parent component is loaded after the child component. For example:

```
// app.vue
<script setup>
import {onMounted} from 'vue'
import ChildVue from './child.vue'
onMounted(() => {
    console.log('app')
})
</script>
<template>
    <ChildVue />
</template>
// child.vue
<script setup>
import {onMounted} from 'vue'
onMounted(() => {
    console.log('child')
})
</script>
```

Running result:

```
child
app
```

If a page has many child components, since the child components are loaded first, the parent component may experience a rather long white screen waiting time. How can we make the child.vue component load asynchronously? Vue3 provides a new defineAsyncComponent method to implement asynchronous components.

## Usage

- **Lazy Loading**: Asynchronous components allow delaying the loading of components until they are needed, which is very important for optimizing initial loading time. By only loading them when they are really needed, we can avoid a performance drop caused by loading all components at once.

- **Large Applications**: When apps have a massive amount of components, synchronous loading of all components might cause very long initial loading time. By using asynchronous components, we can divide the code of the application and load components as needed, thus improving the performance of the application.

- **Conditional Loading**: Some components are only needed to be loaded under specific conditions, such as components that need to be loaded only under certain routes or components that need to be loaded after the user performs certain operations. By using asynchronous components, we can dynamically load components according to conditions, reducing unnecessary initial loading.

**The implementation principle is as follows:**

- Use defineAsyncComponent to define asynchronous components, which returns a component option object that wraps the asynchronous loading logic.

- Create a placeholder component for rendering before the asynchronous component is loaded.

- Handle the asynchronous loading logic in the rendering function of the placeholder component, returning a Suspense component to show the loading status.

- Use import() to dynamically import the component module, which returns a Promise object.
Create a rendering function for the asynchronous component, which is created according to the options object of the component.

- Replace the placeholder component, rendering the asynchronous component in the position of the placeholder component.

## Asynchronous Component

The concept of asynchronous components is not new in Vue3, Vue2 also has asynchronous components. The difference is that Vue2 uses functions to create. Creating asynchronous components in Vue3 is also relatively simple, mainly through the defineAsyncComponent method to create an asynchronous component, and then use the asynchronous component when needed. defineAsyncComponent mainly has two ways:

### Creating with a function that loads Promise

defineAsyncComponent receives a load function that returns Promise. We know that import is static by default. If we use import to dynamically import modules, then a Promise will be returned. That is, we can directly use import in the load function of defineAsyncComponent to dynamically import a module.

Please don’t use dynamic import without necessity.

```
// app.vue
<script setup>
import {onMounted, defineAsyncComponent } from 'vue'
const AsyncChild = defineAsyncComponent(() => import('./child.vue'))
onMounted(() => {
    console.log('app')
})
</script>
<template>
    <AsyncChild />
</template>
```

We can also create asynchronous components using new Promise() ourselves:

```
// app.vue
<script setup>
import {onMounted, defineAsyncComponent } from 'vue'
import Child from './child.vue'
const AsyncChild = defineAsyncComponent(() => (new Promise((resolve, reject) => resolve(Child))))
onMounted(() => {
    console.log('app')
})
</script>
<template>
    <AsyncChild />
</template>
```

Results of their operation:

```
app
child
```

From this, we can see that the parent component is executed before the child component, so the child component has become an asynchronous component.

### Creating with Object

The creation method of dynamic import is simple and effective, but sometimes we has some special requirements, such as knowing the load status of asynchronous components, including loading, loading failure, etc. If it is created by dynamic import, this effect cannot be achieved. Therefore, we need a more advanced way to create: passing a special object.

**Syntax:**

```
const AsyncComp = defineAsyncComponent({
  // Loading function, a promise needs to be returned. You can use dynamic import or new Promise() yourself
  loader: () => import('./Foo.vue'),
  // Component used when loading asynchronous components, this component will be displayed when the asynchronous component is loading, if the asynchronous component loads very fast, the loading component may not appear
  loadingComponent: LoadingComponent,
  // Delay time before loading component is shown, default is 200ms
  delay: 200,
  // Component to be displayed after loading fails, can be tested with Promise's reject
  errorComponent: ErrorComponent,
  // If a timeout time limit is provided and it times out
  // It will also display the error component you configured here. The default value is: Infinity
  timeout: 3000
})
```

Syntax description is not complicated, let’s give some more examples:

First, we create two loading status components, which are also simple, just a sentence:

```
// LoadingComp.vue
<template>
  <div>Loading...</div>
</template>
// ErrorComp
<template>
  <div>Something went wrong...</div>
</template>
```

**Component Loading Failed**

We use new Promise() to test. If you are using dynamic import, I have not found a suitable way to test.

```
// app.vue
<script setup>
import {ref, onMounted, defineAsyncComponent } from 'vue'
import LoadingComp from './LoadingComp.vue'
import ErrorComp from './ErrorComp.vue'
const AsyncChild = defineAsyncComponent({
    loader: () => (new Promise((resolve, reject) => reject())),
    loadingComponent: LoadingComp,
    delay: 200,
    errorComponent: ErrorComp,
    timeout: 2000
})
onMounted(() => {
    console.log('app')
})
let isShowAsyncComp = ref(false)
const loader = () => {
  isShowAsyncComp.value = true
}
</script>
<template>
    <button @click="loader">Load Asynchronous Component</button>
    <AsyncChild v-if="isShowAsyncComp" />
</template>
```

Running result:

Click on the Load Asynchronous Component button, the error message Something went wrong… will appear, because we returned a resolve() Promise in the loader load function.

**Component Loading**

Since it’s an asynchronous component, it’s usually those components that need time-consuming loading. So we can simulate a time-consuming loading component by manually using setTimeout. This case still uses new Promise().

```
// app.vue
...
const AsyncComp = defineAsyncComponent({
  loader: () => (new Promise((resolve, reject) => setTimeout(() => {
    resolve(directiveVue)
  }, 1000))),
  loadingComponent: LoadingComp,
  delay: 200,
  errorComponent: ErrorComp,
  timeout: 2000
})
```

We assume that asynchronous components need to consume 1000ms. Before this, it should be loading.

### Global Asynchronous Components

Just like registering global components, you can also register global asynchronous components.

app.component('AsyncPage',defineAsyncComponent(()=>import('@/views/UiAsyncComponent/Page.vue')))

### Using with Suspense

In Vue3, we can solve this problem by using Suspense. Suspense allows us to define some placeholders to display content before asynchronous components are loaded, and it can automatically switch to the real component after the asynchronous components are loaded.

Suspense is an experimental feature and it is not guaranteed to become a stable version. If you must use this feature, it is recommended to wait until the version is stable. Therefore, our examples are limited to how to use it, and for the principle, you need to check the official website yourself:

```
// app.vue
<!-- <AsyncComp v-if="isShowAsyncComp" /> -->
  <Suspense v-if="isShowAsyncComp">
    <template #default>
      <AsyncComp />
    </template>
    <template #fallback>
      <p>Suspense Loading...</p>
    </template>
  </Suspense>
```

Suspense can only handle loading status and load asynchronous components itself, and cannot handle errors. Of course, you can choose to capture with the onErrorCaptured() hook, or specify an error component when creating asynchronous components with defineAsyncComponent.

Once again, Suspense is just an experimental feature. If it’s in a real project, it is not recommended to use Suspense.

## Summary

This article details how to create and use asynchronous components, as well as handle the loading status of asynchronous components. A simple summary:

- Asynchronous components use the defineAsyncComponent method to create, and need to pass in a load function that returns as Promise.

- There are two ways to create asynchronous components: Promise, Object.

- Asynchronous components can be used with Suspense. Please note that this is currently an experimental feature and is fraught with uncertainty.

**Note**: When configuring routes in Vue-Router, although you can also load route components in a similar asynchronous loading mechanism, you should not use defineAsyncComponent.
