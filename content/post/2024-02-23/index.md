---
title: 'Understanding Teleport in Vue3'
date: 2024-02-23T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*GgMh0JyvKDIrI7ahnm8uiQ.jpeg"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: "Teleport can help us “transport” part of the component template to any location in the DOM, not just limited to the parent component’s template."
---

## Introduction

In Vue2, we often encounter such a problem: we want to render a component to a specific location in the DOM structure, but due to the componentization of Vue, we can only render the component into the parent component’s template. This may cause styling problems or z-index level issues for some special scenarios (such as full-screen modal boxes, notifications, tips, etc.).

To solve this problem, Vue3 introduces the Teleport component. Teleport can help us “transport” part of the component template to any location in the DOM, not just limited to the parent component’s template. This way, we can control the rendering location of the component more flexibly.

## Usage

### Props

- to — This props is requested.Specify the target container, must be a valid query selector or HTMLElement

- disabled —This props is not requested. When the value is true, the content is kept in its original location instead of the specified target location. This attribute can be changed dynamically.

Teleport’s basic usage is very simple. All you need to do is add the `<teleport>` tag in your template and use the to attribute to specify where you want to transport the template. Such as:

```
<teleport to="#end-of-body">
  <div>This will be teleported to #end-of-body</div>
</teleport>
```
In the above code, the `<div>` tag and its content will be “transported” into the DOM element with the id end-of-body.

### Disable Teleport

In some scenarios, you may need to disable `<Teleport>` depending on the situation. For instance, we want to render a component as a popup on the desktop, but as an inline component on mobile. We can handle these two different situations by dynamically passing a disabled prop to `<Teleport>`.

```
<Teleport :disabled="isMobile">
…
</Teleport>
```

The isMobile state can be dynamically updated based on different results of CSS media query.

### Multiple Teleports Share Targets

A reusable modal box component may have multiple instances at the same time. For such scenarios, multiple `<Teleport>` components can mount their content onto the same target element, and the order is simply sequentially appended, with the later mounted ones placed later under the target element.

For example like this:

```
<Teleport to="#modals">
  <div>A</div>
</Teleport>
<Teleport to="#modals">
  <div>B</div>
</Teleport>
```

The rendering result is:

```
<div id="modals">
  <div>A</div>
  <div>B</div>
</div>
```

## Scenarios and Precautions for Using Teleport

The most common use scenarios for Teleport include but are not limited to: modal boxes, alert boxes, notifications, etc. In these scenarios, we usually want the component to be rendered to a specific location in the DOM, rather than being limited to the parent component’s template.

There are a few things to note when using Teleport:

The target location of Teleport must be a valid DOM element. You can use CSS selectors to specify it.

`<Teleport>` only changes the rendered DOM structure, it does not affect the logical relationship between components. That is, if `<Teleport>` contains a component, the component always maintains a logical parent-child relationship with the component that uses `<teleport>`. The passed props and triggered events will also work as usual. This also means that injections from parent components will also work as expected, child components will be nested under parent components in the Vue Devtools, instead of being placed where the actual content is moved.

When `<Teleport>` mounts, the transported to target must already exist in the DOM. Ideally, this should be an element outside the entire Vue application DOM tree. If the target element is also rendered by Vue, you need to ensure that this element is mounted before mounting `<Teleport>`.

## Example

### Use with Dynamic Components

Teleport can not only render static content, but also be combined with dynamic components, to achieve the dynamic rendering of different components in different locations. We will demonstrate this technique through an example.

```
<template>
  <div>
    <h1>Advanced Example of Teleport and Dynamic Components</h1>
    <Teleport :to="teleportTarget">
      <component :is="getCurrentComponent"></component>
    </Teleport>
    <button @click="toggleComponent">Switch Component</button>
  </div>
</template>

<script>
import { ref, defineComponent, Teleport } from 'vue';

export default defineComponent({
  setup() {
    const currentComponent = ref('ComponentA');
    const teleportTarget = ref(null);

    const toggleComponent = () => {
      currentComponent.value = currentComponent.value === 'ComponentA' ? 'ComponentB' : 'ComponentA';
    };

    const getCurrentComponent = () => () => import(`./components/${currentComponent.value}.vue`);

    return {
      currentComponent,
      teleportTarget,
      toggleComponent,
      getCurrentComponent,
    };
  },
});
</script>
```

### Multi-level Nested Components

Teleport is very useful when dealing with multi-level nested components. For example, when a component is nested in multiple levels of parent components but needs to be rendered in a DOM element at another level, Teleport can easily solve this problem.

```
<template>
  <div>
    <h1>Teleport and Multi-level Nested Component Example</h1>
    <ParentComponent>
      <Teleport :to="teleportTarget">
        <NestedComponent />
      </Teleport>
    </ParentComponent>
    <div ref="teleportTarget"></div>
  </div>
</template>

<script>
import { ref, defineComponent, Teleport } from 'vue';
import ParentComponent from './components/ParentComponent.vue';
import NestedComponent from './components/NestedComponent.vue';

export default defineComponent({
  components: {
    ParentComponent,
    NestedComponent,
    Teleport,
  },
  setup() {
    const teleportTarget = ref(null);

    return {
      teleportTarget,
    };
  },
});
</script>
```

### Conditional Rendering Usage

In certain cases, we may need to dynamically render Teleport component based on conditions, or use condition rendering within Teleport component. Below are examples of some common advanced usage techniques.

```
<template>
  <div>
    <h1>Advanced Examples of Teleport and Conditional Rendering</h1>
    <Teleport v-if="shouldRender" :to="teleportTarget">
      <template v-if="condition">
        <p>Conditional Rendering Example</p>
      </template>
      <template v-else>
        <p>Alternative Rendering Example</p>
      </template>
    </Teleport>
    <button @click="toggleRender">Toggle Rendering</button>
  </div>
</template>

<script>
import { ref, defineComponent, Teleport } from 'vue';

export default defineComponent({
  setup() {
    const shouldRender = ref(true);
    const condition = ref(true);
    const teleportTarget = ref(null);

    const toggleRender = () => {
      shouldRender.value = !shouldRender.value;
    };

    return {
      shouldRender,
      condition,
      teleportTarget,
      toggleRender,
    };
  },
});
</script>
```

These advanced techniques and complex scenario examples provide you with a more in-depth guide to using Teleport. By understanding these examples and applying them to your projects, you will be better able to take advantage of the flexibility and powerful features of the Teleport component.

Please note that the Teleport component has many other features and uses, such as rendering to the body tag, using dynamic rendering targets, etc. In actual development, based on the specific needs, you can further explore more features and applications of Teleport.
