---
title: 'Understanding reactive, isReactive and shallowReactive in Vue3'
date: 2024-03-20T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/0*Lf_gf2uitG3fuTvg.jpg"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: "The reactivity system in Vue 3 is the core of its data binding, which allows automatic updating of the DOM when there is a change in reactive state...."
---

The reactivity system in Vue 3 is the core of its data binding, which allows automatic updating of the DOM when there is a change in reactive state. Besides ref, there are also reactive methods for setting up reactive objects.

The function of reactive is to convert a plain object into a reactive object. It recursively turns all properties of the object into reactive data. What it returns is a Proxy object.

---

## reactive

**Parameters**

The parameters for reactive can only be objects, arrays, or collection types like Maps and Sets. This means that properties within the object will automatically trigger view updates when they are changed.

**usage**

```
<script setup>
  import { reactive } from 'vue'

  const person = reactive({
    name: 'Echo',
    age: 25,
  })
  
  console.log(person.name); // 'Echo'
  person.age = 28;          // change property
  console.log(person.age);  // 28
  console.log(person);

</script>
```

The person in browser console will show:

![](https://miro.medium.com/v2/resize:fit:850/format:webp/1*34oSS7ZEbtUlpTs8J7nNwg.jpeg)

As we can see, what’s printed out is a Proxy object. This means that reactive implements reactivity based on the ES6 Proxy.

However, there are several characteristics of Proxy that we need to be aware of:

The reactive() function returns a Proxy of the original object, which is not equal to the original object.

```
<script setup>
  import { reactive } from 'vue'

  const raw = {}
  const proxy = reactive(raw)
  
  console.log(proxy === raw) // false

</script>
```

When the data within the original object changes, it will affect the proxy object; likewise, when the data inside the proxy object changes, the corresponding original data will also change.

```
<script setup>
  import { reactive } from 'vue'

  const obj = {
    count: 1
  }
  const proxy = reactive(obj);
  
  proxy.count++;
  console.log(proxy.count); // 2
  console.log(obj.count);   // 2
</script>
```

Now think, when the data within the original object changes, it will affect the proxy object; and when the data inside the proxy object changes, the corresponding original data will change as well, which is inevitable. However, in actual development, should we operate on the original object or the proxy object?

The answer is: the proxy object, because the proxy object is reactive.

The official recommendation is the same: only the proxy object is reactive, and changing the original object will not trigger updates. Therefore, the best practice using Vue’s reactivity system is to only use the proxy version of the object you declare.

To ensure consistent access to the proxy, calling reactive() on the same original object will always return the same proxy object, and calling reactive() on an existing proxy object will return the proxy itself:

```
<script setup>
  import { reactive } from 'vue'

  const raw = {}
  const proxy1 = reactive(raw)
  const proxy2 = reactive(raw)
  
  console.log(proxy1 === proxy2) // true
  console.log(reactive(proxy1) === proxy1) // true
</script>
```

**Note**: that a reactive object defined with reactive will deeply observe properties at every level, affecting all nested attributes. In other words: a reactive object will also deeply unwrap any ref properties while maintaining reactivity.

```
<script setup>
  import { reactive } from 'vue'

  let obj = reactive({
    name: 'Echo',
    a: {
      b: {
        c: 1
      }
    }
  })

  console.log("obj: ", obj)
  console.log("obj.name: ", obj.name)
  console.log("obj.a: ", obj.a)
  console.log("obj.a.b: ", obj.a.b)
  console.log("obj.a.b.c: ",obj.a.b.c)
</script>
```

![](https://miro.medium.com/v2/resize:fit:802/format:webp/1*oeKW7zP3yBicKXpbTn7fsw.jpeg)

The returned object and the objects nestled within it will all be wrapped with Proxy.

## isReactive

isReactive() is used to check if an object is a reactive proxy created by reactive.

**Usage**:

isReactive() is used to detect if something is of the reactive type; it returns true if it is, otherwise, it returns false.

```
<script setup>
    const refObj = ref({
      name: 'xxxx'
    })
    const refObj1 = ref(3)
    const reactiveObj = reactive({
      name: 'wnxx'
    })
    const refObj2 = shallowRef({
      name: 'xxxx'
    })
    const reactiveObj1 = readonly(
      reactive({
        name: 'xxxx'
      })
    )
    const reactiveObj2 = shallowReactive({
      name: 'xxxx'
    })
    const refObj3 =  readonly(
      ref({
        name: 'xxxx'
      })
    )

    console.log(isReactive(refObj.value)) // true
    console.log(isReactive(refObj1)) // false
    console.log(isReactive(reactiveObj)) // true
    console.log(isReactive(refObj2.value)) // false
    console.log(isReactive(reactiveObj1)) // true
    console.log(isReactive(reactiveObj2)) // true
    console.log(isReactive(refObj3.value)) // true
</script>
```

Note: If the proxy was created with readonly() but wraps another proxy created with reactive, it will also return true.

## shallowReactive

shallowReactive() is used to create a reactive object. shallowReactive() tracks the reactivity of its own properties but ignores reactivity updates of nested properties within the object. Any properties that use ref will not be automatically unwrapped by the proxy.

**Usage**:

```
import { shallowReactive} from 'vue' 

setup(){ 
    let person = shallowReactive({ 
        name:'xxxx', 
        address:{
          adName: 'adxxxx'
        }
    }) 
}
```

**Note**:

- Shallow data structures should be used only for root-level state in components. Avoid nesting them within deeply reactive objects as they create trees with inconsistent reactivity which can be difficult to understand and debug.

- When dealing with both shallow and deep data in object data, use it with caution!!!

- shallowReactive() is typically used when we need to create a reactive object, but do not want its nested properties to be reactively updated as well. This API can be employed in such scenarios.

## The difference between ref and reactive

- ref is mainly used for creating a single reactive piece of data, whereas reactive is used for creating an object with multiple reactive properties.

- For defining variables with primitive types (such as numbers and booleans), ref is recommended; for reactive wrapping of objects or arrays, reactive is the way to go.

- When using reactive data in a template, there’s no need to access ref type data with .value, instead, use the variable name directly. However, when using reactive type data, you directly use the property name of the object.

- reactive will recursively convert all properties of an object to reactive data.

- ref returns an object constructed by the RefImpl class, whereas reactive returns a reactive proxy of the original object using Proxy.

- There’s another difference in how watch is used to listen to ref and reactive: the approach differs, which I will explain in detail below.

1. Using watch to listen to reactive data defined by ref (in the case of primitive data types)

```
<script setup>
  import { ref, watch } from 'vue'

  let count = ref(0)
  watch(count, (newValue, oldValue) => {
    console.log(`new：${newValue}，old：${oldValue}`)
  })
  const changeCount = () => {
    count.value += 10;
  }
</script>

<template>
  <div class="main">
    <p>count: {{ count }}</p>
    <button @click="changeCount">count</button>
  </div>
</template>
```

```
new: 10, old: 0
new: 20, old: 10
new: 30, old: 20
new: 40, old: 30
new: 50, old: 40
new: 60, old: 50
new: 70, old: 60
new: 80, old: 70
new: 90, old: 80
```

When the data being watched is a primitive type defined by ref, the callback function of watch will be executed whenever the data changes.

2. Using watch to listen to reactive data defined by ref (in the case of reference data types)

```
<script setup>
  import { ref, watch } from 'vue'

  let count = ref({ num: 0 })
  watch(count, () => {
    console.log(`count changed`)
  })
  const changeCount = () => {
    count.value.num += 10;
  }
</script>

<template>
  <div class="main">
    <p>count: {{ count }}</p>
    <button @click="changeCount">count</button>
  </div>
</template>
```

When clicking the “count” button, the value of count on the interface updates, but the console does not print out any output. This situation occurs because watch is not performing a deep watch on count. However, it should be noted that the DOM is able to update at this time.

In order to perform a deep watch, it only requires adding an appropriate parameter, { deep: true }.

```
<script setup>
  import { ref, watch } from 'vue'

  let count = ref({ num: 0 })
  watch(count, () => {
    console.log(`count changed`)
    },
    { deep: true }
  )
  const changeCount = () => {
    count.value.num += 10;
  }
</script>

<template>
  <div class="main">
    <p>count: {{ count }}</p>
    <button @click="changeCount">count</button>
  </div>
</template>
```

When we listen directly to count.value without deep watching, the result is the same as with deep watching.

```
<script setup>
  import { ref, watch } from 'vue'

  let count = ref({ num: 0 })
  watch(count.value, () => {
    console.log(`count changed`)
  })
  const changeCount = () => {
    count.value.num += 10;
  }
</script>

<template>
  <div class="main">
    <p>count: {{ count }}</p>
    <button @click="changeCount">count</button>
  </div>
</template>
```

3. Using watch to listen to reactive data defined by reactive.

```
<script setup>
  import { reactive, watch } from 'vue'

  let count = reactive({ num: 0 })
  watch(count, () => {
    console.log(`count changed`)
  })
  const changeCount = () => {
    count.num += 10;
  }
</script>

<template>
  <div class="main">
    <p>count: {{ count }}</p>
    <button @click="changeCount">count</button>
  </div>
</template>
```

When using the watch function to listen to reactive data, it is not necessary to add the deep property in order to perform a deep watch.
