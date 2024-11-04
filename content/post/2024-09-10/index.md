---
title: 'Did you know that Vue3 Components Can “Pause” Rendering？'
date: 2024-09-10T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/0*u0epgmNovwsaszzN.png"
categories: ["Story","Fiction", "Life"]
tags: ["Story","Fiction", "Life"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

## Introduction

Sometimes we want to render a component only after fetching data from the server. Here are some methods to achieve this:

- Request data in the parent component and use `v-if` to render the child component only after receiving the data, passing it via props.

- Request data in the `onMounted` lifecycle of the child component and use `v-if` in the template to control rendering.

Both methods have drawbacks. The ideal solution is to handle data fetching inside the child component and “pause” rendering until the data is ready.

## Example

### Data Request in Parent Component

Father:

```
<template>
  <Child v-if="user" :user="user" />
  <div v-else>
    <p>loading...</p>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";
import Child from "./Child.vue";
const user = ref(null);
async function fetchUser() {
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({
        name: "John",
        phone: "xxxxxxxxxx",
      });
    }, 2000);
  });
}
onMounted(async () => {
  user.value = await fetchUser();
});
</script>
```

Child:

```
<template>
  <div>
    <p>userName：{{ user.name }}</p>
    <p>phone：{{ user.phone }}</p>
  </div>
</template>

<script setup lang="ts">
const props = defineProps(["user"]);
</script>
```

This method involves fetching `user` data in the parent component and passing it to the child, showing a loading message in the meantime. However, it places the data-fetching logic in the parent component, which isn't ideal.

### Data Request in Child Component’s onMounted

Father:

```
<template>
  <Child />
</template>

<script setup lang="ts">
import Child from "./Child.vue";
</script>
```

Child:

```<template>
  <div v-if="user">
    <p>userName：{{ user.name }}</p>
    <p>phone：{{ user.phone }}</p>
  </div>
  <div v-else>
    <p>loading...</p>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from "vue";

const user = ref(null);

async function fetchUser() {
  // Simulate data fetching from the server
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({
        name: "Jhon",
        phone: "xxxxxxxxxx",
      });
    }, 2000);
  });
}

onMounted(async () => {
  user.value = await fetchUser();
});
</script>
```

We placed the data request in `onMounted`, which triggers the first rendering of the child component during initialization. At this point, `user` is still `null`, so we have to use `v-if="user"` in the template to prevent displaying the child component's content and render a `loading` message with `v-else`.

When data is retrieved from the server and the reactive variable `user` is reassigned, it triggers a page re-render. This leads to a second rendering where the child component's content is displayed.

As seen above, this approach causes the child component to render twice. Additionally, placing the `loading` logic inside the child component increases its complexity. Thus, this approach is not ideal.

The perfect solution is to “pause” rendering of the child component during `fetchUser`, using `fallback` to display a loading page. This `loading` logic doesn't need to be inside the child component—it appears automatically during the "paused" period. Once the server data request is complete, the child component begins to render, and the `loading` page is automatically removed.

## Solution

The first method’s drawback is that the child component only renders after receiving data, but the data request logic is placed in the parent component. Ideally, all logic should be encapsulated within the child component.

The second method’s drawback is that the child component actually renders once during initialization before data is retrieved from the server. We have to control this with `v-if` in the template to prevent rendering the child component's content. After receiving data from the server, the child component renders a second time, displaying its content. This approach clearly renders the child component twice.

Is there a perfect solution where the data retrieval logic is inside the child component, and rendering is “paused” until the data request completes?

The answer is yes! Vue 3’s `Suspense` component combined with using `await` at the top level of `setup` can perfectly fulfill this requirement!

Here’s the official introduction to Suspense:

> `<Suspense>` is a built-in component that coordinates the handling of async dependencies in the component tree. It allows us to wait for multiple nested async dependencies to resolve and render a loading state while waiting.

For more on using Suspense, check out this [article](https://medium.com/stackademic/understanding-suspense-in-vue3-af0dd74f04c9).

The `Suspense` component supports two slots: `#default` and `#fallback`. If the `#default` slot contains async components, it first renders the content in `#fallback`. Once the async component loads, it replaces the `#fallback` content with the async component's content.

If our child component is async, `Suspense` can help us achieve the desired functionality.

`Suspense` can automatically render a `loading` state using the `#fallback` slot during the loading of async child components. Once the async component loads, it renders the child component's content for the first time.

In this case, the problem is how to turn our child component into an asynchronous component?

The answer is already given by Vue’s official documentation. If a component uses `await` at its top level in `<script setup>`, it will become an asynchronous component. We simply need to use `await` in the top level of our child component to request data from the server.

Here’s what the transformed code looks like:

Father:

```
<template>
  <Suspense>
    <AsyncChild />
    <template #fallback>loading...</template>
  </Suspense>
</template>

<script setup lang="ts">
import AsyncChild from "./AsyncChild.vue";
</script>
```

The `Suspense` component is used in the parent component, passing 2 slots to this component. The `#default` slot is for the asynchronous child component AsyncChild, the default slot can be used without adding `#default` to the top of the element.

And with the `#fallback` slot, the asynchronous child component AsyncChild will not be rendered first during the asynchronous child component loading process. Instead, the `loading` in the `#fallback` slot will be rendered first, and then the `loading` will be automatically replaced with the content of the child when the asynchronous child finishes loading.

Child:

```
<template>
  <div>
    <p>userName：{{ user.name }}</p>
    <p>phone：{{ user.phone }}</p>
  </div>
</template>

<script setup lang="ts">
import { ref } from "vue";

const user = ref(null);
user.value = await fetchUser();

async function fetchUser() {
  // Simulate data fetching from the server
  return new Promise((resolve) => {
    setTimeout(() => {
      resolve({
        name: "Jhon",
        phone: "xxxxxxxxxx",
      });
    }, 2000);
  });
}
</script>
```

We use `await` in the top level of `<script setup>` and then assign the value obtained by `await` to the user variable. After using `await` at the top level, the child component becomes an asynchronous component and is loaded only after `await fetchUser()` is executed, i.e., after getting the data from the server.

And since we use `Suspense` in the parent component, we don't render the child component until it's loaded, i.e., until we get the data from the server (which is equivalent to “pausing” the rendering of the child component). Instead, it renders the `loading` in the `#fallback` slot, and waits until it gets the data from the server before the asynchronous subcomponent is loaded. Only then will the subcomponent be rendered for the first time, and the `loading` will be replaced with the rendered content of the subcomponent.

Since we got the value of `user` from the server the first time we rendered the subcomponent, `user` is no longer `null`, so we don't have to use `v-if=“user”` at the top of the template, even though we have to read `user.name` in the `template`.

After the `Suspense` + top-level `await` modification, when rendering the parent's `Suspense` and finding out that its child has an asynchronous component, the rendering of the child will be “paused” and the `loading` component will be rendered automatically instead.

The child component uses `await` at the top level of `setup` to wait for the data request from the server, when the data is gotten from the server then the child component is considered to be loaded, then the first rendering will be done, and the content of loading will be replaced by the content rendered in the child component automatically.

And `Suspense` also supports multiple asynchronous sub-components to get data from the server, when these asynchronous sub-components all get data from the server, then it will automatically replace the `loading` with the content rendered by these asynchronous sub-components.

The `Suspense` component is still experimental and should be used with caution in production environments, but Nuxt3 is already using Suspense internally.

## Principle

When rendering a subcomponent, `Suspense` does not immediately execute the `render` function of the asynchronous subcomponent when it realizes that the subcomponent is an asynchronous component. Instead, it adds a flag called `deps` to indicate that the default subcomponent is currently an asynchronous component, and pauses rendering the asynchronous subcomponent.

Since the asynchronous subcomponent is a Promise, you can add the `.then()` method after loading the Promise of the asynchronous subcomponent, and only in the `.then()` method will you go ahead and continue rendering the asynchronous subcomponent.

Currently the asynchronous subcomponent has paused rendering, and then it will go and read the `deps` token. If the `deps` flag is `true`, the asynchronous subcomponent has paused rendering, and the `loading` component in the `fallback` slot will be rendered to the page.

When the asynchronous subcomponent finishes loading it triggers the `.then()` method of the Promise, which continues rendering the asynchronous subcomponent. The `.then()` method executes the `render` function of the asynchronous subcomponent to generate the virtual DOM, and then generates the real DOM based on the virtual DOM, and finally replaces the content of the `fallback` slot that was rendered on the page with the content of the real DOM generated by the asynchronous component.
