---
title: 'Understanding defineModel in Vue3'
date: 2024-03-08T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1274/format:webp/1*XNUMO2PM_SwVgmdGARQtCQ.png"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: "It can simplify the two-way binding between parent and child components and is the currently officially recommended two-way binding implementation."
---

With the release of `vue3.4` version, `defineModel` has also officially become a regular version. It can simplify the two-way binding between parent and child components and is the currently officially recommended two-way binding implementation.

---

## How to implement two-way binding in the past

Everyone should know that `v-model` is just syntactic sugar. It actually defines the `modelValue` attribute and listens for the `update:modelValue` event for the component, so we had to implement two-way data binding before. You need to define a `modelValue` attribute for the subcomponent, and when you want to update the modelValue value in the subcomponent, you need to `emit` send out a `update:modelValue` event. Pass the new value as the second field.

Let’s look at a simple example. The code of the parent component is as follows:

```
<template>
  <CommonInput v-model="inputValue" />
</template>

<script setup lang="ts">
import { ref } from "vue";

const inputValue = ref();
</script>
```

The code for the subcomponent is as follows:

```
<template>
  <input
    :value="props.modelValue"
    @input="emit('update:modelValue', $event.target.value)"
  />
</template>

<script setup lang="ts">
const props = defineProps(["modelValue"]);
const emit = defineEmits(["update:modelValue"]);
</script>
```

The above example should be very familiar to everyone. It’s how we used to implement two-way binding with `v-model`. But there is a problem, the input box actually supports direct use of `v-model`, but we did not use `v-model` here, instead we `added` value attribute and `input` event to the input box.

The reason is that starting from Vue2, it has been a one-way data flow, and the value in the `props` cannot be directly modified in the child components. Instead, an event should be thrown from the child component, and the parent component should listen to this event, and then modify the variable passed to the `props` in the parent component. If we directly add `v-model = 'props.modelValue’` to the input box here, it is actually modifying the `modelValue` directly in the `props` of the child component. Due to the reason of one-way data flow, Vue does not support the direct modification of `props`, so we need to write the code in the above way.

## Use defineModel to implement two-way data binding

The code of the parent component is the same as before, as follows:

```
<template>
  <CommonInput v-model="inputValue" />
</template>

<script setup lang="ts">
import { ref } from "vue";

const inputValue = ref();
</script>
```

The code for the subcomponent is as follows:

```
<template>
  <input v-model="model" />
</template>

<script setup lang="ts">
const model = defineModel();
model.value = "xxx";
</script>
```

In the above example, we directly bind the return value of `defineModel` to the input box using `v-model` , without defining the `modelValue` attribute and monitoring `update:modelValue` is a `ref` . We can modify the value of the `model` variable in the child component, and the `inputValue` in the parent component The value of the variable is also updated synchronously, so that two-way binding can be achieved.

Now here comes the question, from Vue2 onward, we’ve been operating with one-way data flow. When the value of the child component is modified, the variable value of the parent component is also changed. Does this not revert back to the two-way data flow of Vue1? Actually it’s not the case, it is still a one-way data flow.

We will briefly explain the implementation principle of `defineModel` in the following.

## Implementation principle

`defineModel` actually defines a variable called `model` within the child component as ref and `modelValue` as props, and it also watches the `modelValue` in the props. When the value of `modelValue` in props changes, it will synchronize and update the value of the model variable.

Moreover, when the value of the `model` variable inside the child component changes, it will `emit` an `update:modelValue` event. Once the parent component receives this event, it will update the corresponding variable value within the parent component.

The implementation principle code is as follows.

```
<template>
  <input v-model="model" />
</template>

<script setup lang="ts">
import { ref, watch } from "vue";

const props = defineProps(["modelValue"]);
const emit = defineEmits(["update:modelValue"]);
const model = ref();

watch(
  () => props.modelValue,
  () => {
    model.value = props.modelValue;
  }
);
watch(model, () => {
  emit("update:modelValue", model.value);
});
</script>
```

After reading the above code, you should understand why the return value of `defineModel` can be directly modified in the child component, and the corresponding variables of the parent component will also be updated synchronously. What we modify is actually the ref variable returned by `defineModel` , rather than directly modifying modelValue in props. The implementation method is still the same as vue3.4 's previous two-way binding, except that `defineModel` macro helps us encapsulate the previous cumbersome code into internal implementation.

In fact, the source code of `defineModel` is implemented using `customRef` and `watchSyncEffect`. I used the examples of ref and watch here, in order to make it easier for everyone to understand the implementation principle of `defineModel`.

## How to define type, default, etc. in defineModel?

Since `defineModel` declares a `prop`, it can also define the `type` and default of `prop`. The specific code is as follows.

```
const model = defineModel({ type: String, default: "20" });
```

In addition to supporting `type` and `default` , required and validator are also supported. The usage is the same as when defining `prop` .

## How to implement multiple v-model bindings in defineModel?

It also supports multiple `v-model` bindings on the parent component. At this time, the first parameter we pass to `defineModel` is not an object, but a string.

```
const model1 = defineModel("count1");
const model2 = defineModel("count2");
```

The code when using v-model in the parent component is as follows:

```
<CommonInput v-model:count1="inputValue1" />
<CommonInput v-model:count2="inputValue2" />
```

We can also define `type` , `default` , etc. in multiple `v-model`

```
const model1 = defineModel("count1", {
  type: String,
  default: "aaa",
});
```

## How to use built-in and custom modifiers in defineModel?

If you want to use the system’s built-in modifiers such as `trim` , the writing method of the parent component is still the same as before:

```
<CommonInput v-model.trim="inputValue" />
```

There is no need to make any modifications to the subcomponent, it is the same as the other `defineModel` examples above:

```
const model = defineModel();
```

`defineModel` also supports custom modifiers. For example, if we want to implement a uppercase custom modifier that changes all letters in the input box to uppercase, we also need to use the built-in trim modifier.

The parent component code is as follows:

```
<CommonInput v-model.trim.uppercase="inputValue" />
```

The subcomponent needs to be written like this:

```
<template>
  <input v-model="modelValue" />
</template>

<script setup lang="ts">
const [modelValue, modelModifiers] = defineModel({
  set(value) {
    if (modelModifiers.uppercase) {
      return value?.toUpperCase();
    }
  },
});
</script>
```

At this time, the first argument we pass to `defineModel` is an object containing `get` and `set` methods. When reading the `modelValue` variable, it will go into the `get` method. When writing to the `modelValue` variable, it will go into the `set` method. If only interception of write operations is needed, then `get` can be omitted.

The return value of `defineModel` can also be destructured into two variables. The first variable is the `ref` object we used for `v-model` binding in the previous examples. The second variable is an object, which contains what modifiers are there. Here we have two modifiers `trim` and uppercase, so the `value` of modelModifiers is:

```
{
  trim: true,
  uppercase: true
}
```

When input is made in an input box, it will go to the `set` method, and then calling `value?.toUpperCase()` can convert the entered letters into uppercase.

## Summary

This article introduces how to use the `defineModel` macro to implement two-way binding and the implementation principle of `defineModel`.

- Calling the defineModel macro within a child component will return a ref object. Inside the child component, this ref object can be directly assigned, and the corresponding variable in the parent component will be modified synchronously.

- Essentially, defineModel defines a ref variable and a corresponding prop within the child component, and then listens to the corresponding prop to keep the value of the ref variable consistent with the corresponding prop. When the value of the ref variable is modified in the child component, an event is thrown to the parent component to update the corresponding variable value, thereby achieving two-way binding.

- By using defineModel({ type: String, default: ‘20’ }) you can define options such as type and default for the prop.

- Using defineModel(‘count’) can implement multiple v-model bindings.

- By destructuring the return value of defineModel(), the modelModifiers modifier object can be obtained. Custom modifiers can be implemented in combination with the “get” and set converter options.
