---
title: 'Understanding KeepAlive in Vue3'
date: 2024-03-26T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/0*SNKB1Duz47DDtnG1.png"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: "KeepAliveis a built-in component in Vue, which is used to cache components in memory to prevent redundant DOM rendering..."
---

## Overview

`KeepAlive` is a built-in component in Vue, which is used to cache components in memory to prevent redundant DOM rendering, thus trading off memory consumption for speed. A common use case is caching components or routes.

- `KeepAlive` is a global component in Vue 3.

- `KeepAlive` itself does not render and will not appear in the DOM nodes, but it is rendered as a vnode. Through the vnode, one can track the cache and keys within `KeepAlive`. Of course, this is only in the development environment; after building the package, these are not exposed in the vnode (this still needs to be confirmed).

- The most important function of `KeepAlive` is to cache components.

- `KeepAlive` updates the component cache using an LRU (Least Recently Used) caching eviction strategy, which can make more efficient use of memory and prevent memory overflow. In the source code, the maximum cache size max is 10. This means that after 10 components, it starts to evict the earliest cached component.

---

## Usage

### Basic

By default, a component instance is destroyed when it gets replaced. This leads to the loss of all its changed state — when the component is displayed again, a new instance with only its initial state will be created.

Assume that there is a group of Tab components on the page, as shown in the following code:

```
<template>
  <Tab v-if="currentTab === 1">...</Tab>
  <Tab v-if="currentTab === 2">...</Tab>
  <Tab v-if="currentTab === 3">...</Tab>
</template>
```

As you can see, depending on the value of the variable `currentTab`, different Tab components will be rendered. When a user frequently switches Tabs, this leads to the constant unloading and rebuilding of the corresponding Tab components. To avoid the performance overhead, you can use the `KeepAlive` component to solve this problem, as shown in the following code:

```
<template>
   <KeepAlive>
      <Tab v-if="currentTab === 1">...</Tab>
      <Tab v-if="currentTab === 2">...</Tab>
      <Tab v-if="currentTab === 3">...</Tab>
   </KeepAlive>
</template>
```

In this way, regardless of how users switch Tab components, there won’t be frequent creation and destruction, which significantly optimizes the response to user operations. This advantage becomes particularly apparent in the context of large components.

### include/exclude

KeepAlive by default caches all internal component instances, but we can customize this behavior through the include and exclude props.

Both of these props can be a comma-separated string, a regular expression, or an array containing these two types:

```
<!-- Comma separated string -->
<KeepAlive include="a,b">
  <component :is="view" />
</KeepAlive>
<!-- Regular expression (use `v-bind`) -->
<KeepAlive :include="/a|b/">
  <component :is="view" />
</KeepAlive>
<!-- Array (use `v-bind`) -->
<KeepAlive :include="['a', 'b']">
  <component :is="view" />
</KeepAlive>
```

It matches based on the component’s `name` option, so if a component wants to be conditionally cached by `KeepAlive`, it must explicitly **declare a name option**.

### Maximum Number of Cached Instances

We can limit the maximum number of component instances that can be cached by passing in a max prop. The behavior of `<KeepAlive>` when a max is specified is akin to an LRU (Least Recently Used) cache: if the number of cached instances is about to exceed the specified maximum number, the cached instance that has not been accessed for the longest time will be destroyed to make space for new instances.

```
<KeepAlive :max="10">
  <component :is="activeComponent" />
</KeepAlive>
```

### Dynamic Removal of Cache

There is no method provided for us to actively delete the cache. For example, if I have a multi-tab management background, when I set up to cache up to ten pages, and then I open ten pages, even after closing these ten pages, the `keep-alive` component still retains these caches. If I open one of these ten pages again, it still uses the cache, and the component does not re-render.

For instance, if we want to refresh a page and there’s no method to actively delete the cache, then we can only add new cache, which might fill up the cache if you refresh a single page ten times.

Clearly, these are situations we don’t want to see. Using the `keep-alive` component is intended to improve performance. From the perspective of the include parameter of the component, it can be used to clear the cache. The approach is as follows: add the component name to include to cache the component; remove the component name to clear its cache. Based on this principle, we can simply encapsulate the code with a hook:

```
// hooks
import { ref, nextTick } from 'vue'

const caches = ref<string[]>([])

export default function useCache () {
  // Add cached components
  function addCache (componentName: string | string []) {
    if (Array.isArray(componentName)) {
      componentName.forEach(addCache)
      return
    }
    
    if (!componentName || caches.value.includes(componentName)) return

    caches.value.push(componentName)
  }

  // Remove cached components
  function removeCache (componentName: string) {
    const index = caches.value.indexOf(componentName)
    if (index > -1) {
      return caches.value.splice(index, 1)
    }
  }
  
  // Instances of removed cached components
  async function removeCacheEntry (componentName: string) {    
    if (removeCache(componentName)) {
      await nextTick()
      addCache(componentName)
    }
  }
  
  return {
    caches,
    addCache,
    removeCache,
    removeCacheEntry
  }
}
```

### Lifecycle of Cached Instances

When a component instance is removed from the DOM but is still kept alive as part of the component tree due to being cached by `<KeepAlive>`, it will enter an inactive state instead of being unmounted.

When a component instance that is part of the cached tree is re-inserted into the DOM, it will be reactivated.

A persisting component can register lifecycle hooks for these two states through `onActivated()` and `onDeactivated()`:

```
<script setup>
import { onActivated, onDeactivated } from 'vue'
onActivated(() => {
  // This is called when initially mounted
  // and each time it's re-inserted from the cache
})
onDeactivated(() => {
  // This is called when it's removed from the DOM, enters the cache,
  // and when the component is unmounted
})
</script>
```

**Note:**

- `onActivated` will also be called during component mount, and `onDeactivated` will also be called when the component is unmounted.

- These two hooks are not only applicable to the root components cached by `<KeepAlive>` but also to descendant components within the cached tree.

### Maintain the scroll position of the cached page

```
// hooks
import { onActivated } from 'vue'
import { onBeforeRouteLeave } from 'vue-router'
let gobalScrollBox = 'html' // Global scroll box
export function configKeepScroll (scrollBox: string) {
  gobalScrollBox = scrollBox
}
export default function useKeepScroll (scrollBox?: string) {
  let pos = [0, 0]
  let scroller: HTMLElement | null
  
  onActivated(() => {
    scroller = document.querySelector(scrollBox || gobalScrollBox)
    if (!scroller) {
      console.warn(`useKeepScroll: not found ${scrollBox || gobalScrollBox} Dom scroll box`)
      return
    }
    scroller.scrollTop = pos[0]
    scroller.scrollLeft = pos[1]
  })
  
  onBeforeRouteLeave(() => {
    if (scroller) {
      pos = [scroller.scrollTop, scroller.scrollLeft]
    }
  })
}
```

### Using with vue-router

```
<template>
  <router-view v-slot="{ Component }">
    <keep-alive>
      <component :is="Component"  v-if="$route.meta.keepAlive"/>
    </keep-alive>
    <component :is="Component"  v-if="!$route.meta.keepAlive"/>
  </router-view> 
</template>
```

```
// router.js
{
  path: "/keepAliveTest",
   name: "keepAliveTest",
   meta: {
       keepAlive: true //Set whether the page needs to use cache
   },
   component: () => import("@/views/keepAliveTest/index.vue")
 },
...
```
