---
title: 'Understanding the various ref in Vue3'
date: 2024-02-02T16:22:38+08:00
draft: true
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*B-rRsR1EO03Ui8Dn7a9XsQ.png"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: "In Vue3, there are many functions related to responsiveness, such as toRef, toRefs, isRef, unref, etc."
---

In Vue3, there are many functions related to responsiveness, such as toRef, toRefs, isRef, unref, etc. Reasonably using these functions can greatly improve efficiency in actual development.

## ref()

Everyone is no stranger to the `ref` API. It is often used in Vue3. Its function is to accept a value and return a responsive object. We can access and modify this value through the `.value` property. In the template, we can omit `.value`, for example in the following code, when the button is clicked, the count on the page will change responsively.

```
<template>
    <div>
        {{ count }}
        <button @click="addCount">+1</button>
    </div>
</template>

<script lang='ts' setup>
import { ref } from "vue"
const count = ref(1)
const addCount = () => {
    count.value++
}
</script>
```

## toRef

`toRef` can create a responsive `ref` based on a property in a responsive object. At the same time, this `ref` is synchronized with the property in the original object. If the value of the original object property changes, the `ref` will change accordingly, and vice versa. It accepts two parameters, one is the corresponding response, and the other is the property value, such as the following code.

```
<template>
    <div>
        {{ count.a }}
        {{ a }}
        <button @click="addCount">+1</button>
    </div>
</template>

<script lang='ts' setup>
import { ref, toRef } from "vue"
const count = ref({
    a: 1,
    b: 2
})
const a = toRef(count.value, 'a')
const addCount = () => {
    a.value++
}
</script>
```

When you click the button to modify the value of a, the a in count will also change. Of course, the count here can also use reactive.

## toRefs

`toRefs` can convert a responsive object into a plain object, and each property of this plain object is a responsive ref.

```
<template>
    <div>
        {{ count.a }}
        {{ countAsRefs.a }}
        <button @click="addCount">+1</button>
    </div>
</template>

<script lang='ts' setup>
import { reactive, toRefs } from "vue"
const count = reactive({
    a: 1,
    b: 2
})
const countAsRefs = toRefs(count)
const addCount = () => {
    countAsRefs.a.value++
}

</script>
```

At this time, the type of `countAsRefs` in the code is

```
{
  a: Ref<number>,
  b: Ref<number>
}
```

Its properties a and b are responsive `ref` objects. Similarly, they are also synchronized with the properties of the original count object.

According to its characteristics, we usually use it to deconstruct a responsive object without losing its responsiveness.

```
import { reactive, toRefs } from "vue";
const count = reactive({
  a: 1,
  b: 2,
});
const { a, b } = toRefs(count);
```

At this point, both a and b are responsive `ref` objects and are synchronized with the original object’s `a` and `b` properties.

## isRef()

As the name suggests, `isRef` is used to determine whether a value is a `ref`.

**Note: it cannot determine whether this value is reactive (isReactive can be used for this purpose).**

```
import { reactive, isRef, ref } from "vue";
const count = ref(1);
const testObj = reactive({
  a: 1,
});
console.log(isRef(count)); //true
console.log(isRef(testObj)); //false
```

## unref()

In fact, it is a syntactic sugar.

```
val = isRef(val) ? val.value : val;
```

If it is a `ref`, it will return its internal value, otherwise, it will return itself. This syntactic sugar shows that it can remove the reactive reference for reactive objects. For instance, if we only want to get a reactive value but do not want it to be reactive, we can use it to dereference. For example:

```
<template>
    <div>
        {{ unRefAsCount }}
        {{ count }}
        <button @click="addCount">+1</button>
    </div>
</template>

<script lang='ts' setup>
import { unref, ref } from "vue"
const count = ref(1)
let unRefAsCount = unref(count)
const addCount = () => {
    count.value++
}
</script>
```

The `unRefAsCount` in the code does not have reactivity.

## shallowRef

From the literal meaning, we can see that it is a shallow ref. What is a shallow ref? The difference with `ref` is that only `.value` is reactive, while deeper properties are not reactive.

```
<template>
    <div>
        {{ shallowObj.a }}
        <button @click="addCount"> +1</button>
    </div>
</template>

<script lang='ts' setup>
import { shallowRef } from "vue"

const shallowObj = shallowRef({
    a: 1
})
const addCount = () => {
    //Does not trigger page updates
    shallowObj.value.a++
}
</script>
```

However, if we change addCount to modify the entire `.value`, it will trigger reactivity.

```
const addCount = () => {
  let temp = shallowObj.value.a;
  temp++;
  shallowObj.value = {
    a: temp,
  };
};
```

## triggerRef

It allows a `shallowRef` to forcefully trigger changes when changes occur in its deep properties. For example, adding `triggerRef` to the above code example, which cannot trigger reactivity.

```
<template>
    <div>
        {{ shallowObj.a }}
        <button @click="addCount"> +1</button>
    </div>
</template>

<script lang='ts' setup>
import { shallowRef, triggerRef } from "vue"

const shallowObj = shallowRef({
    a: 1
})

const addCount = () => {
    shallowObj.value.a++
    //Add 'triggerRef' to force changes.
    triggerRef(shallowObj)
}
</script>
```

## customRef

As the name suggests, it is a custom `ref`. We can explicitly track the reactive changes of a certain value through `customRef`. It accepts a function, which takes track and trigger as parameters, and returns an object with get and set methods. For example, encapsulate a custom reactive object `myRef` and control it so that it only triggers reactivity when the value is less than 4.

```
<template>
    <div>
        {{ count }}
        <button @click="addCount"> +1</button>
    </div>
</template>

<script lang='ts' setup>
import { customRef } from "vue"
const myRef = (value: number) => {
    const customValue = customRef((track, trigger) => {
        return {
            get() {
                //Inform Vue that it needs to track changes in subsequent content, and this can be freely controlled.
                track()
                return value
            },
            set(newValue) {
                console.log(newValue);//myRef.value=xxx
                //Adding a 'trigger' triggers reactivity, notifying Vue to update the page. Here you have the freedom to control whether to add 'trigger'.
                if(value<4)  trigger()
                value = newValue
            }
        }
    })
    return customValue
}

const count = myRef(0)
const addCount = () => {
    count.value++
}
</script>
```

When count is greater than 4, it loses its reactivity.

## Summary

This article gives a detailed introduction to the various uses of ‘ref’ in Vue3, which includes:

- **ref()**: Takes a value and returns a reactive object. The .value property can be used to access and modify this value.

- **toRef(obj, key)**: Based on a property of a reactive object, it creates a reactive ref, and this ref remains in sync with the property in the original object.

- **toRefs(obj)**: Converts a reactive object into a regular object, where each property of the regular object is a reactive ref.

- **isRef(value)**: Determines whether a value is a ref object.

- **unref(value)**: Used to remove reactive references.

- **shallowRef(value)**: Creates a shallow ref, only the value property is reactive, the deep property is not reactive.

- **triggerRef(ref)**: Forces shallow ref to trigger reactive when changes occur.

- **customRef(factory)**: Customizes the ref object, and can explicitly track the reactive changes of a value.
