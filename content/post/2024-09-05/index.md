---
title: 'Learn These New Features in Vue 3.5 Now!'
date: 2024-09-05T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*nSM03v1PRpXL2yqKIHEPYA.png"
categories: ["Story","Fiction", "Life"]
tags: ["Story","Fiction", "Life"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

## Introduction

On September 3, 2024, the official Vue team released Vue.js 3.5, a stable version containing no breaking changes but including internal improvements and useful new features. Let's explore some interesting changes.

## Reactive Props Destructuring

Before Vue 3.5, you used `toRef()` to destructure props reactively:

```
<script setup lang="ts">
import { toRef } from 'vue'

const props = defineProps<{
  count: number
}>()

// Reactive destructuring of count property
const count = toRef(props, 'count')

const log = () => {
  console.log(count.value)
}
</script>
```

In Vue 3.5, you can directly destructure reactively:

```
<script setup lang="ts">

// Vue 3.5 allows direct reactive destructuring
const { count } = defineProps<{
  count: number
}>()

const log = () => {
  console.log(count)
}
</script>
```

Access to a destructured variable like count is automatically compiled into props.count by the compiler, so it is tracked on access.

## Default Prop Values

Before Vue 3.5, default props values were written like this:

```
// With TypeScript generics (at compile time)
const props = withDefaults(
  defineProps<{
    count: number
  }>(),
  {
    count: 0
  }
)

// With JavaScript (at runtime)
const props = defineProps({
  count: {
    type: Number,
    default: 2
  }
})
```

In Vue 3.5, default prop values can be written like this:

```
// ts
const { count = 1 } = defineProps<{
  count: number
}>()

// js
const { count = 2 } = defineProps({
  count: String
})
```

You can assign default values directly like in JavaScript object destructuring, which is pretty nice!

## Obtaining Template References with useTemplateRef()

In Vue 3, you can use `ref` for template references to access the DOM and child components, but `ref` can be confusing. For example, is a `ref` variable reactive data or a DOM element? Vue 3.5 introduces `useTemplateRef`, which solves these issues perfectly.

```
<template>
  <div class="list" ref="listEl">
    <div ref="itemEl" class="item" v-for="item in list" :key="item">
      {{ item }}
    </div>
  </div>
</template>

<script setup lang="ts">
import { useTemplateRef, onMounted } from 'vue'

const list = [1,2,3]

const listRef = useTemplateRef('listEl')
const itemRef = useTemplateRef('itemEl')

onMounted(() => {
  console.log(listRef.value)  // div.list
  console.log(itemRef.value)  // Proxy(Array) {0: div.item, 1: div.item, 2: div.item}
})
</script>
```

When a template-bound ref has multiple elements with the same name, u`seTemplateRef()` returns an array, as shown with the `v-for` rendered list.

## Teleport Component Improvements

`<Teleport>` Specific usage can be found in [this article](https://medium.com/@kxming/understanding-teleport-in-vue3-42943b104d3e)

The `<Teleport>` component has a known limitation where its target element must exist when the component is mounted. In Vue 3.5, `<Teleport>` introduces a defer prop to mount it after the current render cycle:

```
<Teleport defer to="#cont">
  <div v-if="open">
    <span>Mounted to div with id 'cont'</span>
    <button @click="open = false">Close</button>
  </div>
</Teleport>
<!-- Container for Teleport content -->
<div id="cont"></div>
<button @click="open = true">Open</button>
```


Since `<div id="cont"></div>` is rendered after the Teleport, you need the defer prop. If `<div id="cont"></div>` is rendered before, defer is not needed.

## Generating Unique Application IDs with useId()

`useId()` is used to generate unique IDs for each application, ensuring stability across server and client renders. It can be used for generating IDs for form elements and accessibility attributes without causing ID conflicts in SSR applications:

```
<script setup>
import { useId } from 'vue'

const id = useId()
</script>

<template>
  <form>
    <label :for="id">Name:</label>
    <input :id="id" type="text" />
  </form>
</template>
```

## 参考引用

For more updates in Vue 3.5, check out the [Announcing Vue 3.5](https://blog.vuejs.org/posts/vue-3-5) link for more information.
