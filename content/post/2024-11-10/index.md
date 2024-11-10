---
title: 'Understanding useId in Vue3'
date: 2024-11-10T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/0*6aTlWcQeK-86tPxH.png"
categories: ["Story","Fiction", "Life"]
tags: ["Story","Fiction", "Life"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

## Introduction

The useId function, newly added in Vue 3.5, is primarily used to generate unique IDs. These IDs are unique within the same Vue application, and each call to useId generates a different ID. This functionality is very useful when handling list rendering, form elements, and accessibility attributes, as it ensures that each element has a unique identifier.

The implementation principle of useId is relatively simple. It generates IDs by accessing the ids property of the Vue instance, which is an array containing a prefix and an incrementing number. Each time useId is called, it retrieves the current number, increments it, and returns the ID. This means that multiple Vue application instances on the same page can avoid ID conflicts by configuring app.config.idPrefix, as each application instance maintains its own ID generation sequence.

## 1. Source Code

```
export function useId(): string {
  const i = getCurrentInstance()
  if (i) {
    return (i.appContext.config.idPrefix || 'v') + '-' + i.ids[0] + i.ids[1]++
  } else if (__DEV__) {
    warn(
      `useId() is called when there is no active component ` +
        `instance to be associated with.`,
    )
  }
  return ''
}
```

- `i.appContext.config.idPrefix`: This is a configuration property obtained from the current component instance to define the prefix for the generated ID. If this prefix exists, it will be used; otherwise, the default 'v' will be used.

- `i.ids[0]`: This is the first element of the ids array on the current component instance, typically an empty string, used as part of the ID generation.

- `i.ids[1]++`: This is the second element of the ids array, a number used for the incrementing part of the ID. The post-increment operator ++ is used here, which means it returns the current value and then increments it. Each time useId is called, this number increases, ensuring that the generated ID is unique.

## 2. Setting the ID Prefix

If you do not want to use the default prefix 'v', you can set it using `app.config.idPrefix`.

```
const app = createApp(App)

app.config.idPrefix = 'vid'
```

## 3.Usage

### 3.1 Unique Identifier for Form Elements

In forms, the `<label>` tag needs to match the for attribute with the corresponding `<input>` tag's id attribute to ensure that clicking the label focuses the input field. Using useId can generate a unique ID for each `<input>` element, ensuring this functionality works correctly. For example:

```
<template>
  <label :for="id">Do you like Vue 3.5?</label>
  <input type="checkbox" :id="id" />
</template>

<script setup>
const id = useId()
</script>
```

### 3.2 Unique Keys in List Rendering

When rendering lists, each item typically requires a unique key to help Vue track the identity of each node for efficient DOM updates. If your list data does not have a unique key, useId can generate a unique key for each item in the list.

```
<template>
  <ul>
    <li v-for="item in items" :key="item.id">
      {{ item.text }}（{{ item.id }}）
    </li>
  </ul>
</template>

<script setup>
const items = Array.from({ length: 10}, (v, k) => { 
    return {
        text: `Text ${k}`,
        id: useId()
    }
})
</script>
```

Result: 

```
·Text 0(vid-1)
.Text 1(vid-2)
·Text 2(vid-3)
·Text 3(vid-4)
·Text 4(vid-5)
.Text 5(vid-6)
.Text 6(vid-7)
·Text 7(vid-8)
.Text 8(vid-9)
·Text 9(vid-10)
```

### 3.3 Avoiding ID Conflicts in Server-Side Rendering (SSR)

Here’s an example of server-side rendering:

```
<template>
  <div>
    <label :htmlFor="id">Do you like Vue3.5?</label>
    <input type="checkbox" name="vue3.5" :id="id" />
  </div>
</template>

<script setup lang="ts">
const id = Math.random();
</script>
```

The above code works fine when rendered on the client side, but will trigger a warning in server-side rendering due to differing ID values, generating a `Hydration attribute mismatch` warning.

Why is an ID generated on the server and then again on the client?

To answer this, we need to understand the server-side rendering (SSR) process:

1. First, a request is made on the server (Node.js environment) to fetch the data needed to render the page.

2. Based on the received data, the server generates the HTML string, generating an ID in this step, called dehydration.

3. The server-generated HTML string is sent to the client (browser).

4. The browser receives this HTML string and renders it as the initial content. However, at this point, click events have not yet been bound to the DOM, so the client needs to render again, generating an ID once more, called hydration.

Since we used `Math.random()` to generate the ID, the IDs generated on the server and client will differ, leading to the warning mentioned above.

With `useId`, resolving this warning becomes simple: just replace `Math.random()` with `useId()`.

This is because `useId` will generate `v-0` during server-side rendering and will still be `v-0` during client rendering.

Some may wonder, didn’t we just say that `useId` increments the number each time it is executed? Shouldn’t the IDs generated on the server and client be different after executing once on the server and then once on the client?

The incrementing part generated by `useId` is maintained in the ids property of the Vue instance. During server-side rendering, a Vue instance is created on the Node.js side. However, a new Vue instance is created in the browser for client rendering, resetting the ids property. Therefore, the IDs generated by useId on the server and client are the same.
