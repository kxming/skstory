---
title: 'Usage of h() in vue3'
date: 2024-08-12T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*GqSbIvTEgHMoyHcuDhB_ZA.png"
categories: ["Story","Fiction", "Life"]
tags: ["Story","Fiction", "Life"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

## Introduction

According to Vue’s official documentation, `h()` it is a rendering function used to create virtual DOM nodes. When building a page with HTML tags in a Vue project, the final result will be converted into a vnode. The `h()` function creates a vnode directly, allowing for more flexibility in constructing component rendering logic and improving performance.

## Syntax

The `h()` rendering function has three parameters, of which the second and third are optional:

- **type**: The type of node to create, which can be an HTML tag, component, or function (functional component).

- **props (optional)**: An object containing properties for the node, passed as a prop.

- **children (optional)**: Child nodes that can be strings, arrays, or other vnode objects.

```
// Basic usage example
function h(
  type: string | Component,
  props?: object | null,
  children?: Children | Slot | Slots
): VNode

// Example usage
h('div', { id: 'foo' }, 'Hello!');
```

Parameter Explanation

1. `type`:

- HTML tag: If there type is a string, it will be parsed as an HTML tag.

- Component: If the type is an object or function, it will be parsed as a Vue component.

- Asynchronous component: The type can also be a function that returns a Promise, which will be resolved to the component.

2. `props`:

- Props are optional parameters used to specify properties for the node.

- When passing props, you should pass an object containing property names and values as a prop.

- You can also pass `null` it as a value for props.

3. `children`:

- Child nodes can be strings, arrays, or other vnode objects.

- If child nodes are an array, each element in the array will be considered a node’s child.

- If a child node is a function, it will be called during rendering and its return value will be used as the child node.

## Example Usage Scenarios

Below are some real-world project examples where you might want to use the `h()` function:

### Creating Virtual DOM Elements

**In an options API component**: when defining the `render` option, you can return a vnode using the `h()` function.

```
// Options API example
import { h } from 'vue'

export default {
  render() {
    return h('div', { className: 'app' }, [
      h('h2', { className: 'title' }, 'I am title'),
      h('p', null, 'I am content')
    ])
  }
}
// or
import { defineComponent } from "vue";
export default defineComponent({
  name: "Jsx",
  render() {
    return <div>I am div</div>;
  },
});
```

**Composition API Components**: When using Composition API components, you can also define a `setup` the function that returns a rendering function.

```
// Composition API example
<script>
import { h } from "vue"

export default {
  setup() {
    // setup is a function that returns another function
    return () => h("div", { class: "app" }, [
      h("h2", { class: "title" }, "I am title"),
      h("p", null, "I am content")
    ])
  }
}
</script>
```

### Dynamic Rendering of Components

As shown, combining `component` for dynamic rendering of components is one of the most common business scenarios. However, many people do not use `h()` to implement this function in such cases. In fact, using `h()` can avoid the overhead of template compilation in this scenario, bringing about performance optimization and making it easier to handle as well.

```
<template>
  <button @click="changeComponent">switch</button>
  <component :is="createComponent()" />
</template>

<script setup lang="ts">
import { ref } from 'vue';
import ComponentA from './ComponentA.vue';
import ComponentB from './ComponentB.vue';

const nowComponent = ref('ComponentA');

function changeComponent() {
  nowComponent.value = nowComponent.value === 'ComponentA' ? 'ComponentB' : 'ComponentA';
}

const createComponent = () => {
  return h(nowComponent.value === 'ComponentA' ? ComponentA : ComponentB);
};
</script>
```

### Creating Functional Components

Using `h()` to create functional components can not only be independently extracted and maintained but also eliminate the need for an additional `.vue` file, which is truly very practical. This approach is particularly suitable for simple UI components, as it can significantly simplify code and improve performance.

```
<template>
  <FunctionalComponent text="Functional component" @click="handleClick" />
</template>

<script setup lang="ts">
import { h, defineEmits } from 'vue';

const FunctionalComponent = (props, context) => {
  return h('div', null, [
    h('p', null, props.text),
    h('button', { onClick: context.emit.bind(context, 'click') }, 'click')
  ]);
};

const emit = defineEmits(['click']);

function handleClick() {
  console.log('handleClick');
}
</script>
```

As shown above, this is a typical method of using functional components, including how to interact with them in parent-child components. The FunctionalComponent receives props and context parameters and uses the `h()` function to construct the page. Additionally, it triggers event handlers in the parent component through `context.emit`.

Note: Props are used to receive attributes passed from parent components. Context is used to access the component context, such as slots and method events.

### Rendering Dynamic Attributes

If a component or tag needs to define some dynamic attributes, using the `h()` rendering function becomes quite convenient.

```
<template>
  <button @click="myVisibility">Switch status</button>
  <component :is="componentWithProps()" />
</template>

<script setup lang="ts">
  import { ref } from 'vue';

  const visible = ref(true);

  function myVisibility() {
    visible.value = !visible.value;
  }

  const componentWithProps = () => {
    return h('div', { class: { visible: visible.value } }, 'div');
  };
</script>
```

As shown above, the `h()` function inside determines whether the vnode has the `visible` class based on the value of `visible`, and implements dynamic styling when the button is clicked. Of course, dynamic classes are just an example; in reality, all kinds of attributes or child components within `h()` can be dynamic and flexible, far surpassing what can be achieved with HTML.

**Note**: The term `vnode` refers to a virtual node, which is a representation of a DOM node in the Vue.js framework.

### Using Slots

When using `h()` with functional components and slots to pass content, it can greatly improve flexibility.

```
<template>
  <SlotComponent>
    <template #default>
      <p>In slot</p>
    </template>
  </SlotComponent>
</template>

<script setup lang="ts">
import { h } from 'vue';

const SlotComponent = (props, context) => {
  return h('div', null, [
    h('p', null, 'In slot:'),
    context.slots.default && context.slots.default()
  ]);
};
</script>
```

As shown above, when referencing the `SlotComponent` component, the contents within its internal `<template>` tag will be passed as the default slot's content to the `SlotComponent`. Additionally, you can use `context.slots.default()` to retrieve and render the content of the default slot. Therefore, when we encapsulate a functional component but have dynamic parts inside, it is particularly suitable for using slots in this way.

Note: Props are used to receive attributes passed from parent components. Context is used to access the component context, such as slots and method events.

### Creating Dynamic Tags

The code below uses `h()` function combined with dynamic components to achieve switching between different HTML tags.

```
<template>
  <button @click="changeTag">Switch</button>
  <component :is="createElement()" />
</template>

<script setup lang="ts">
import { ref } from 'vue';

const tag = ref('div');

function changeTag() {
  tag.value = tag.value === 'div' ? 'section' : 'div';
}

const createElement = () => {
  return h(tag.value, null, 'dynamic');
};
</script>
```

As shown above, it simply determines which HTML tag’s VNode to return based on the value of tag. When business logic involves dynamic changes to tags, this approach is beneficial and avoids the use of `v-if` producing a large amount of redundant code.

**Note**: VNode refers to a virtual node, which is a representation of a DOM node in the Vue.js framework.
