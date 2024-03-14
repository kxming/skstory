---
title: 'Understanding v-model in Vue3'
date: 2024-02-02T16:22:38+08:00
draft: true
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*N1yhl_KuYBBBkB5FsmmdMQ.png"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: "In Vue3, there are many functions related to responsiveness, such as toRef, toRefs, isRef, unref, etc."
---

`v-model` is a very important built-in directive in Vue, which creates two-way bindings on form input elements or components. These elements include:

- `<input>`

- `<select>`

- `<textarea>`

- `components`

`v-model` is actually a syntax sugar for the value attribute and the input event

```
<input type="text" :value="iptVal" @input="$event => iptVal = $event.target.value" />
<!-- v-model -->
<input type="text" v-model="iptVal" />
```

We usually use the `v-model` directive to complete data binding when developing forms, and this directive can also be used on components.

## Bind a Single Attribute

### Basic Binding

**Taking the custom component CustomInput as an example**

```
<script setup>
    const txt = ref('');
 </script>
 
 <template>
  <CustomInput v-model="txt" />
 </template>
```

`v-model` will be expanded into the following form

```
<CustomInput
  :modelValue="txt"
  @update:modelValue="newValue => txt = newValue"
/>
```

The `CustomInput` component needs to do two things internally:

- Bind the value attribute of the internal native `<input>` element to the `modelValue` prop

- When the native input event triggers, trigger an `update:modelValue` custom event with a new value

Here is the corresponding code:

```
<template>
    <input :value="modelValue" @input="$emit('update:modelValue', $event.target.value)" />
</template>

<script setup>
  const props = defineProps({
    'modelValue': String,
  })
  const emit = defineEmits(["update:modelValue"])
</script>
```

Some people may find this writing way too cumbersome, and it will make the tag code lengthy.

Another way to implement `v-model` in the component is to use a writable computed property with both a getter and a setter.

**Binding with computed**

When using computed properties, the get method should return modelValue prop, and the set method should trigger the corresponding event

```
<template>
  <input v-model="value" />
</template>

<script setup>
  const value = computed({
    get() {
      return props.modelValue
    },
    set(value) {
      emit("update:modelValue", value)
    }
  })
</script>
```

This way of writing can simplify the properties in the tag and make the logic clear

Binding a single attribute can be easily done with v-model, but what if multiple attributes need to be bound bidirectionally?

## Use v-model to bind multiple attributes

By default, `v-model` uses `modelValue` as prop on a component, and `update:modelValue` as the corresponding event

But we can change these names by specifying a parameter for `v-model`

```
<template>
    <CustomInput v-model:first-name="first" v-model:last-name="last" />
</template>
```

In the same way, you can use two ways to bind, but the prop has changed from the original modelValue to the parameter name passed in, and the corresponding event has also changed to `update:parameter` name.

```
<template>
  <input
    type="text"
    :value="firstName"
    @input="$emit('update:firstName', $event.target.value)"
  />
  <input
    type="text"
    :value="lastName"
    @input="$emit('update:lastName', $event.target.value)"
  />
</template>

<script setup>
const props = defineProps({
  firstName: String,
  lastName: String,
})
// Use it in computed
const emit = defineEmits(['update:firstName', 'update:lastName'])
</script>
```

## Binding an Object

In a complex component, if multiple fields need to be bound in two ways. Using the method mentioned above can be a bit cumbersome.

Here are two ways to bind an object in two ways:

Define the parent component searchBar as a complex form component:

```
<template>
    <searchBar v-model="modelValue" />
</template>

<script setup>
import { ref } from "vue"
const modelValue = ref({
  keyword: "123",
  selectValue: "",
  options: [
    {
      label: "all",
      value: ""
    },
    {
      label: "a1",
      value: "1"
    },
    {
      label: "a2",
      value: "2"
    },
  ]
})
</script>
```

Then, in the searchBar component, we receive modelValue and define the type as Object:

```
<template>
  <div>
    <!-- <input type="text" v-model="modelValue.keyword"> can achieve two-way binding -->
    <input type="text" 
      :value="modelValue.keyword"
      @input="handleKeywordChange"
    >
    <select v-model="modelValue.selectValue">
      <option v-for="o in modelValue.options" :key="o.value" :value="o.value">
        {{ o.label }}
      </option>
    </select>
  </div>
</template>

<script lang="ts" setup>
const props = defineProps({
  modelValue: {
    type: Object,
    default: () => ({})
  }
})
const emit = defineEmits(["update:modelValue"]);
// For example, with input
const handleKeywordChange=(val)=>{
  emit("update:modelValue",{
    ...props.modelValue,
    keyword:val.target.value
  })
}
</script>
```

If an object is passed in, as explained in the comments, `<input type="text" v-model="modelValue.keyword">`. Although this can directly perform bidirectional binding, it will disrupt the one-way data flow.

It’s the same as the emit trigger event above, but the passed data has become an object.

Despite emit allowing bidirectional binding, it is too cumbersome. The following describes a more elegant way of writing, which can be said to be a trick — computed + proxy.

If you use computed binding, you might write this kind of code:

```
<template>
      <input type="text" v-model="model.keyword">
 </template>
 
<script lang="ts" setup>
const model = computed({
  get() {
    return props.modelValue
  },
  set(value) {
    // console.log(value) // Found no print
     emit("update:modelValue", {
      ...props.modelValue,
       keyword: value
     })
  }
})
<script>
```

But when you input, you will find that the setter is not triggered because computed will do a layer of proxy, the proxy object has not changed.

If you want to trigger the setter, see the following picture:

```
// It only changes this way
 model.value = {
   keyword:"asdfad"
 }
```

This method cannot trigger the setter, so it cannot bind in both directions, what should we do?

**Return a proxy object in the getter!**

**Return a proxy object in the getter!**

**Return a proxy object in the getter!**

Since the properties of the proxy object are consistent with the properties of the proxied object, we use proxy to wrap the original object.

Then the `v-model` is bound to the object after the proxy, if the properties of the proxy object change, it will trigger the set method in the proxy object, at which point we can trigger emit.

```
const model = computed({
  get() {
    return new Proxy(props.modelValue, {
      set(obj, name, val) {
        emit("update:modelValue", {
          ...obj,
          [name]: val
        })
        return true
      }
    })
  },
  set(value) {
    emit("update:modelValue", {
      ...props.modelValue,
      keyword: value
    })
  }
})
```

## Modifiers

We know that `v-model` has some built-in modifiers, such as .trim, .number and .lazy.

In some scenarios, we may want the `v-model` of a custom component to support custom modifiers.

Let’s create a custom modifier capitalize that automatically converts the first letter of the string value bound by `v-model` to uppercase:

```
<CustomInput v-model.capitalize="txt" />
```

We added the capitalize modifier, which will be automatically passed into modelModifiers prop:

```
<template>
  <input :value="modelValue" @input="emitValue" />
</template>

<script setup>
  const props = defineProps({
    modelValue: String,
    modelModifiers: {
      default: () => ({})
    }
  })
  const emitValue = (e) => {
    let value = e.target.value;
    // use the modifier
    if (props.modelModifiers.capitalize) {
      value = value.charAt(0).toUpperCase() + value.slice(1)
    }
    emit('update:modelValue', value)
  }
</script>
```

## Problem Background:

In the basics section, we’ve already explained how to encapsulate a component with a customized `v-model`. However, in actual development, using `@input` and `:value` to bind our values in the subcomponent is somewhat cumbersome. Is there a simpler way?

We often want to directly bind the `v-model` to the subcomponent which needs bidirectional binding:

```
<!-- subcomponent -->
<input type="text" v-model="xxx" />
```

Given this, in receiving the pass value from the parent component in the subcomponent, what should xxx bind to? Directly bind it to `props.modelValue`?

```
<!-- subcomponent -->
<input type="text" v-model="props.modelValue"/>
```

An error will occur:

```
⚠️reactivity.esm-bundler.js:512 Set operation on key "modelValue" failed: target is readonly.
```

This is because props is a readonly value (`isReadonly(props) === true`), so we can't use it directly.

Therefore, we need an intermediate value to bind the `v-model`.

### Method One: Relay Through watch

We can bind the `v-model` to an internal variable and use watch to monitor it and synchronize the data of `props.xxx`:

```
<!-- subcomponent -->
<template>
  <input type="text" v-model="proxy" />
</template>

<script setup>
import { ref, watch } from "vue";
const emit = defineEmits();
const props = defineProps({
  modelValue: String,
});
const proxy = ref(props.modelValue);
watch(
  () => proxy.value,
  (v) => emit("update:modelValue", v)
);
</script>
```

Sometimes, we may perform bidirectional binding to an object or array. In this case, we can use watch's deep option to deeply monitor and synchronize proxy:

```
watch(
  () => proxy.value,
  (v) => emit("update:modelValue", v),
  {deep: true}
);
```

As `props.modelValue` may have a default value passed in, we can add the immediate option so that the proxy is assigned the default value as soon as the component is created;

**Method Two: Get and Set with computed**

You can also use the get and set provided by computed to synchronize data:

```
const proxy = computed({
  get() {
    return props.modelValue;
  },
  set(v) {
    emit("update:modelValue", v);
  },
});
```

**Ultimate: Encapsulate v-model Hooks**

First, let's encapsulate the watch method as a hook:

```
<!-- subcomponent -->
<template>
  <input type="text" v-model="proxy" />
</template>

<script setup>
import { ref, watch, computed } from "vue";
const emit = defineEmits();
const props = defineProps({
  modelValue: String,
});
const proxy = ref(props.modelValue);
watch(
  () => proxy.value,
  (v) => emit("update:modelValue", v)
);
</script>
```

In the subcomponent, we bind an internal value proxy to the input via `v-model` and initialize the proxy variable with the value of `props.modelValue` (`ref(props.modelValue`);

In watch, we monitor the binding value proxy on the input. When the value changes due to input, we dispatch the `emit('update:modelValue', v)` event to pass the changed value to the external component dynamically.

After extracting common logic:

```
// useVmodel1.js
import { ref, watch } from "vue";
export function useVmodel(props, emit) {
  const proxy = ref(props.modelValue);
  watch(
    () => proxy.value,
    (v) => emit("update:modelValue", v)
  );
  return proxy;
}
```

The simplest hooks are encapsulated.

```
<template>
  <input type="text" v-model="proxy" />
</template>
<script setup>
import { ref, watch, computed } from "vue";
import { useVmodel } from "./hooks/useVmodel1";
const emit = defineEmits();
const props = defineProps({
  modelValue: String,
});
const proxy = useVmodel(props, emit);
</script>
```

Continue the encapsulation

Considering the following points, continue the encapsulation:

- emit can not be passed, for a simpler call syntax.

- Events such as multiple `v-model:test1`, `emit("update:xxxx")` need to extract the event name xxxx.

Through the `getCurrentInstance` method provided by vue3, we can get the current component instance. As modelValue can be overridden, extract it into a variable:

```
//useVmodel2.js
import { ref, watch, getCurrentInstance } from "vue";
export function useVmodel(props, key = "modelValue", emit) {
  const vm = getCurrentInstance();
  const _emit = emit || vm?.emit;
  const event = `update:${key}`;
  const proxy = ref(props[key]);
  watch(
    () => proxy.value,
    (v) => _emit(event, v)
  );
  return proxy;
}
```

Good. Now, we can call our hooks in a simpler way:

```
<!-- subcomponent childModel -->
<template>
  <input type="text" v-model="modelValue" />
  <input type="text" v-model="test" />
</template>

<script setup>
import { useVmodel } from "./hooks/useVmodel2";
const emit = defineEmits();
const props = defineProps({
  modelValue: String,
  test: String,
});
const modelValue = useVmodel(props);
const test = useVmodel(props, "test");
</script>
<!-- parent component -->
<template>
  <Model v-model="modelValue" v-model:test="test" />
</template> 
<script setup>
import { ref, watch } from "vue";
import Model from "./childModel.vue";
const modelValue = ref("");
const test = ref("");
</script>
```
