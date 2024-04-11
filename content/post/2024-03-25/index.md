---
title: 'Understanding v-memo in Vue3'
date: 2024-03-25T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/0*mHgmXlGtuB6aLKdr.jpg"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: "Memoize a sub-tree of the template. Can be used on both elements and components. The directive expects a fixed-length array of dependency..."
---

## Overview

The official definition of `v-memo` is as follows:

Memoize a sub-tree of the template. Can be used on both elements and components. The directive expects a fixed-length array of dependency values to compare for the memoization. If every value in the array was the same as last render, then updates for the entire sub-tree will be skipped.

Simply put, what `v-memo` does is similar to our existing computed properties, except that v-memo’s target is the DOM.

This new directive will cache the DOM segment it controls, and if a specific value changes, it will just run the update and re-render. These values are set by us manually.

```
  <div v-memo="[valueA, valueB]">
    ...
  </div>
```

---

## Usage

`v-memo` accepts an array of dependencies. If the array changes, the DOM corresponding to `v-memo`, including its children, will be re-rendered. Conversely, if the dependencies array remains unchanged, even if the whole component is re-rendered, updates to the DOM corresponding to `v-memo` and its children will be skipped.

Additionally, the dependencies array can accept one or multiple values, like `v-memo=”[valueOne, valueTwo]”`, and also expressions such as `v-memo=”myValue === true”`.

### Example 1: Empty Array

If an empty array is passed in, it will never re-render and will always use the cached result from the first render, similar to v-once.

```
<template>  
    <div v-memo="[]">{{ data }}</div>
    <!-- equal to -->
    <div v-once>{{ data }}</div>
</template>
```

### Example 2: Variable

```
<template>
  <div v-memo="[name]">
    <div>{{ name }}</div>
    <div>{{ age }}</div>
    <div>{{ message }}</div>
  </div>
  <button @click="handleBtnClick">change</button>
</template>
<script setup>
import { ref } from 'vue';

const name = ref('loftyamb')
const age = ref(24)

const handleBtnClick = () => {
  age.value++;
  // Changes to the age value will not be updated on the page unless the name changes.
}
</script>
```

In certain scenarios, when the business logic is complex, manually controlling the overall updates can enhance performance. This can be very useful if we need precise control over when a large component is re-rendered.

### Example 3: Using with v-for

```
<div v-for="item in list" :key="item.id" v-memo="[item.id === selected]">
  <p>ID: {{ item.id }} - selected: {{ item.id === selected }}</p>
  <p>...more child nodes</p>
</div>
```

If `v-memo` is not used in the code above, every change to the selected variable would cause a complete re-render of the list. The caching provided by the new directive allows only the rows where the expression `item.id === selected` changes to update, that is, when an item is selected or deselected.

## Summary

The use of `v-memo` can be summarized as follows:

- It is not recommended to use `v-memo` if it depends on an empty list.

- In certain scenarios, when the business logic is complex, manually controlling the overall updates can enhance performance.

- If we need to control the re-rendering time of a large component, this is very helpful.

- Optimization for rendering large lists.
