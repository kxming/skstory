---
title: '9 Ways of Component Communication in Vue3'
date: 2024-06-09T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*VVlkja3-kKwDW4l6BeVOYw.jpeg"
categories: ["Code","Javascript"]
tags: ["Code","Javascript"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

## Overview

- props / emit

- provide / inject

- Pinia

- expose / ref

- attr

- v-model

- mitt.js

- Slots

- Teleport

---

## Props/Emit

Parents pass data to children using props, and children communicate back to parents using events.

### Parents pass data to children

```
// Parent.vue sending
<child :msg2="msg2"></child>
<script setup lang="ts">
    import child from "./child.vue"
    import { ref, reactive } from "vue"
    const msg2 = ref<string>("This is the message 2 sent to the child component")
    // Or for complex types
    const msg2 = reactive<string>(["This is the message 2 for the descendant component"])
</script>

// Child.vue receiving
<script setup lang="ts">
    // No need to import, directly use
    // import { defineProps } from "vue"
    interface Props {
      msg1: string
      msg2: string
    }
    const props = withDefaults(defineProps<Props>(), {
      msg1: '',
      msg2: '',
    })
    console.log(props) // { msg2: "This is the message 2 for the descendant component" }
</script>
```

Note:

If the parent component uses the setup() method, and the child component uses the script setup syntax, it won’t receive the properties from the parent’s data, it can only receive the properties passed in the parent’s setup function.

If the parent component uses the script setup syntax sugar and the child component uses the setup() method, the child can receive properties from both the parent’s data and setup function through props. However, if the child component aims to receive properties in its setup, it can only receive properties from the parent’s setup function, not from the data properties.

### Children pass data to parent

```
// Child.vue dispatch
<template>
    // Method one
    <button @click="emit('myClick')">Button</button>
    // Method two
    <button @click="handleClick">Button</button>
</template>
<script setup lang="ts">
    
    // Method one Suitable for Vue3.2 version, no need to import
    // import { defineEmits } from "vue"
    // Corresponding to method one
    const emit = defineEmits(["myClick","myClick2"])
    // Corresponding to method two
    const handleClick = ()=>{
        emit("myClick", "This is the message sent to the parent component")
    }
    
    // Method two Not suitable for Vue3.2 version, useContext() is deprecated
    import { useContext } from "vue"
    const { emit } = useContext()
    const handleClick = () => {
        emit("myClick", "This is the message sent to the parent component")
    }
</script>

// Parent.vue response
<template>
    <child @myClick="onMyClick"></child>
</template>
<script setup lang="ts">
    import child from "./child.vue"
    const onMyClick = (msg: string) => {
        console.log(msg) // This is the message received by the parent component
    }
</script>
```

---

## Provide/Inject

This mechanism is used for developing a dependency injection from parent to any of its descendant components, not necessarily direct child components.

- `provide`: Allows us to specify the data or

- `inject`: Receive the data we want to add to this component in any descendant component, regardless of how deeply the component is nested, it can be used directly.

```
// Parent.vue
<script setup>
    import { provide } from "vue"
    provide("name", "Jhon")
</script>

// Child.vue
<script setup>
    import { inject } from "vue"
    const name = inject("name")
    console.log(name) // Jhon
</script>
```

---

## Pinia

`Pinia` is a brand new Vue state management library designed as a replacement for `Vuex`.

```
// main.ts
import { createPinia } from 'pinia'
createApp(App).use(createPinia()).mount('#app')

// /store/user.ts
import { defineStore } from 'pinia'
export const userStore = defineStore('user', {
    state: () => {
        return { 
            count: 1,
            arr: []
        }
    },
    getters: { ... },
    actions: { ... }
})

// Page.vue
<template>
    <div>{{ store.count }}</div>
</template>
<script lang="ts" setup>
import { userStore } from '../store'
const store = userStore()
// deconstruction
// const { count } = userStore()
</script>
```

---

## expose/ref

References can be used for parent components to access child component instances or elements directly.

```
// Child.vue
<script setup>
    // Method 1 Not suitable for Vue 3.2 version, useContext() has been deprecated in this version
    import { useContext } from "vue"
    const ctx = useContext()
    // Exposing properties and methods, etc.
    ctx.expose({
        childName: "This is a property of the child component",
        someMethod(){
            console.log("This is a method of the child component")
        }
    })
    
    // Method 2 Suitable for Vue 3.2 version, no need to import
    // import { defineExpose } from "vue"
    defineExpose({
        childName: "This is a property of the child component",
        someMethod(){
            console.log("This is a method of the child component")
        }
    })
</script>

// Parent.vue  Note ref="comp"
<template>
    <child ref="comp"></child>
    <button @click="handlerClick">Button</button>
</template>
<script setup>
    import child from "./child.vue"
    import { ref } from "vue"
    const comp = ref(null)
    const handlerClick = () => {
        console.log(comp.value.childName) // Obtaining the property exposed by the child component
        comp.value.someMethod() // Calling the method exposed by the child component
    }
</script>
```

---

## attrs

Contains a collection of non-prop attributes from the parent scope, excluding class and style.

```
// Sending from Parent.vue
<child :msg1="msg1" :msg2="msg2" title="3333"></child>
<script setup>
    import child from "./child.vue"
    import { ref, reactive } from "vue"
    const msg1 = ref("1111")
    const msg2 = ref("2222")
</script>

// Receiving in Child.vue
<script setup>
    import { defineProps, useContext, useAttrs } from "vue"
    // No need to import defineProps in version 3.2, use directly
    const props = defineProps({
        msg1: String
    })
    // Method 1 Not suitable for Vue3.2 version as useContext() has been deprecated
    const ctx = useContext()
    // If msg1 was not received as prop, it would be { msg1: "1111", msg2:"2222", title: "3333" }
    console.log(ctx.attrs) // { msg2:"2222", title: "3333" }
    
    // Method 2 Suitable for Vue3.2 version
    const attrs = useAttrs()
    console.log(attrs) // { msg2:"2222", title: "3333" }
</script>
```

---

## v-model

Supports two-way data binding for multiple pieces of data

```
// Parent.vue
<child v-model:key="key" v-model:value="value"></child>
<script setup>
    import child from "./child.vue"
    import { ref, reactive } from "vue"
    const key = ref("1111")
    const value = ref("2222")
</script>

// Child.vue
<template>
    <button @click="handlerClick">Button</button>
</template>
<script setup>
    
    // Method 1 Not suitable for Vue 3.2 version as useContext() has been deprecated
    import { useContext } from "vue"
    const { emit } = useContext()
    
    // Method 2 Suitable for Vue 3.2 version, no need to import
    // import { defineEmits } from "vue"
    const emit = defineEmits(["key","value"])
    
    // Usage
    const handlerClick = () => {
        emit("update:key", "New key")
        emit("update:value", "New value")
    }
</script>
```

---

## mitt.js

In Vue3, EventBus for cross-component communication is no longer available, but there’s now a replacement called mitt.js, which is based on the same principle as EventBus.

```
// mitt.js
import mitt from 'mitt'
const mitt = mitt()
export default mitt;

// component A
<script setup>
import mitt from './mitt'
const handleClick = () => {
    mitt.emit('handleChange')
}
</script>

// component B 
<script setup>
import mitt from './mitt'
import { onUnmounted } from 'vue'
const someMethed = () => { ... }
mitt.on('handleChange',someMethed)
onUnmounted(()=>{
    mitt.off('handleChange',someMethed)
})
</script>
```

---

## Slots

Slots allow parent components to control part of the content in child components. They are useful for creating reusable and flexible component templates.

### Default Slots

```
// Parent.vue
<FancyButton>
  Click me! <!-- slot content -->
</FancyButton>

// Child.vue
<button class="fancy-btn">
  <slot></slot> <!-- slot outlet -->
</button>
```

### Named Slots

Named slots are a categorization based on the default slot, which can be understood as matching contents to their corresponding placeholders.

```
// Parent.vue
<template>
  <Child>
    <template v-slot:monkey>
      <div>monkey</div>
    </template>

    <button>Click me!</button>
  </Child>
</template>

// Child.vue
<template>
  <div>
    <!-- default slots -->
    <slot></slot>
    <!-- named slots -->
    <slot name="monkey"></slot>
  </div>
</template>

```

### Scoped Slots

The content of a slot cannot access the state of the child component. However, in some scenarios, the content of the slot may want to use data from both the parent component’s sphere and the child component’s sphere. To achieve this, we need a way for the child component to provide some data to the slot at the time of rendering.

```
// Parent.vue
<template>
  <!-- v-slot="{scope}" is used to receive data passed up from the child component -->
  <!-- :list="list" passes the list to the child component -->
  <Child v-slot="{scope}" :list="list">
    <div>
      <div>Name: {{ scope.name }}</div>
      <div>Occupation: {{ scope.occupation }}</div>
      <hr>
    </div>
  </Child>
</template>

<script setup>
import { ref } from 'vue'
import Child from './components/Child.vue'

const list = ref([
  { name: 'Jhon', occupation: 'Thundering'},
  ...
])
</script>

// Child.vue
<template>
  <div>
    <!-- Using :scope="item" to return each item -->
    <slot v-for="item in list" :scope="item" />
  </div>
</template>

<script setup>
const props = defineProps({
  list: {
    type: Array,
    default: () => []
  }
})
</script>
```
