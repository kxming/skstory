---
title: 'The Details About Vue3 that You Didn’t Know'
date: 2024-01-25T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*2VUijywcHU32qLVrQ7BFbA.png"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

## Why is ref recommended over reactive?

`Reactive` has some limitations:

- **Limited value types**: It can only be used for object types (objects, arrays, and collection types such as Map, Set). It cannot hold primitive types like string, number, or boolean.

- **Unable to replace the entire object**: Because Vue’s `reactive` tracking is implemented through property access, we must always maintain the same reference to the reactive object. This means we cannot easily replace a reactive object, because this would lose the reactive connection with the initial reference;

- **Not friendly to deconstructive operations**: When we deconstruct the primitive type properties of a reactive object into local variables, or pass the properties to a function, we will lose the reactive connection.

## Ref Unwrapping

Only the top-level ref properties will be unwrapped in the context of template rendering. Unlike reactive objects, a `ref` will not be unwrapped when it is accessed as an item in a reactive array or a native collection type (like Map).

## Timing Issue with watchPostEffect

By default, the listener callbacks created by users will be called before the Vue component update. This means the DOM accessed in the listener callback will be in the state before Vue’s update. If you wish to access the DOM in the listener callback after Vue’s update, you need to specify the flush: ‘post’ option.

```
watchEffect(
  (onCleanup) => {
    console.log(document.getElementById('pel'));
    console.log('WatchEffect: Count changed:', state.count);
    // console.log('WatchEffect: pel changed:', pel.value);
    onCleanup(() => {
      console.log('WatchEffect: onCleanup');
    });
  },
  {
    flush: 'pre',
  },
);
```

Through testing, the difference between the two is that when using ‘pre’, the webpage gets null when it first fetches DOM from the document. But with ‘post’, it’s able to fetch the DOM.

**Return Value of watchEffect:**

Both watchEffect and watch return a stop function. Once this function is executed, the listening will stop.

## Parsing Native HTML as Vue Component

Certain HTML elements have restrictions on the types of elements placed within them, such as `<ul>`, `<ol>`, `<table>`, and `<select>`. Correspondingly, certain elements will only be displayed when placed within specific elements, such as `<li>`, `<tr>`, and `<option>`.
This can cause problems when using components with such restricted elements. For example:

```
<table>
  <blog-post-row></blog-post-row>
</table>
```

The custom component <blog-post-row> will be ignored as invalid content, thus causing errors in the final rendered output. We can use the special is attribute as a solution:

```
<table>
  <tr is="vue:blog-post-row"></tr>
</table>
```

When used on native HTML elements, the value of ‘is’ must be prefixed with ‘vue:’ to be parsed as a Vue component. This is necessary to avoid confusion with native custom built-in elements.

## DefineProps is a macro declaration in Vue 3, what is a macro declaration?

Macro declaration is a common concept in programming languages, usually referring to the declaration of a macro in the code using specific syntax. Macros can be understood as substitution rules for code snippets, which are expanded or replaced with actual code during compilation.

Macro processing refers to the process of dealing with macros during the compilation stage. When a macro declaration appears in the program code, the compiler will replace the usage of the macro with actual code during the compilation stage based on the definition of the macro. This process is similar to performing text replacement in the code.

In the context of Vue 3, `defineProps` is a macro declaration. It is used to declare the attributes received in the child component and transform these attributes into data with reactivity. The process of macro processing takes place during the compilation stage, and when Vue compiles the code of the child component, it will transform the attributes into reactive attributes according to the declaration of `defineProps`.

Here is a more specific explanation:

- **Macro Declaration**: In Vue 3, `defineProps` is a type of macro declaration. You use `defineProps` in the setup function of your child component to declare the attributes your component receives, telling the Vue compiler that these attributes should have reactivity.

- **Macro Processing**: During the compilation stage, the Vue compiler processes the child component’s code. When it encounters a place where `defineProps` is used, the compiler transforms the attribute declaration into a reactive attribute definition, giving it the ability to automatically track changes at runtime.


## v-bind Binding Object

If you want to pass all the properties of an object as props, you can use `v-bind` without an argument, i.e., use `v-bind` instead of :prop-name. For example, here is a post object:

```
const post = {  id: 1,  title: 'My Journey with Vue'}
```

And the following template:

```
<BlogPost v-bind="post" />
```

However, this is actually equivalent to:

```
<BlogPost :id="post.id" :title="post.title" />
```

## onMounted call stack synchronicity

When onMounted is invoked, Vue automatically registers the callback function on the currently being initialized component instance. This means these hooks should be registered synchronously when a component is initializing. For example, don’t do this:

```
setTimeout(() => {
  onMounted(() => {
    // ...
  })
}, 100)
```

Please note that this does not mean that calls to onMounted have to be placed within the lexical context of `setup()` or `<script setup>`. `onMounted()` can also be called from an external function, provided the call stack is synchronous and ultimately originates from `setup()`.

## Suspense and Asynchronous Components

How to determine asynchronous components:

- Components defined by the defineAsyncComponent function.

- Components with an asynchronous setup() hook. This also includes

- Components with top-level await expressions when using `<script setup>`.

Three events of Suspense:

- The ‘pending’ event is triggered when entering the suspended state.

- The ‘fallback’ event is triggered when the fallback slot content is displayed.

- The ‘resolve’ event is triggered when the default slot has finished fetching new content.

The execution sequence is as follows: `pending -> fallback -> resolve`

What is suspension?

- Suspension recognizes that the components within Suspense have asynchronous dependencies. It waits for these to complete and then reaches a ‘completed’ state.

- If no asynchronous dependencies are encountered during the initial rendering, then it will not enter the suspended state and `<Suspense>` will directly complete.

Timeout parameter in Suspense:

- As the official documentation describes it

- When a fallback happens, the backup content is not presented immediately. Instead, `<Suspense>` displays the previous #default slot content while it waits for new content and asynchronous dependencies to complete. This can be controlled with a timeout prop: After waiting longer than the timeout for new content to render, `<Suspense>` switches to display the backup content. If the timeout value is set to 0, the backup content will be displayed immediately upon the replacement of the default content.

What does this mean? For example, if I had already completed loading an asynchronous component for the first time and the content in default had changed and needed to load asynchronous dependency resources again, say it took 3 seconds to complete, and I set the timeout to 1000 ms, then the original default content would stay on the page for 1000 ms, then render the content in the fallback slot. When the asynchronous task is completed, the default content would be displayed again.

```
<Suspense
      @pending="pending"
      @fallback="fallback"
      @resolve="resolve"
      :timeout="1000"
    >
      <AsyncComp v-if="show"></AsyncComp>
      <div v-else>AsyncComp else </div>
      <template #fallback>
        <div>loading...</div>
      </template>
</Suspense>
```

Asynchronous Components

```
<template>
  <div>sleep</div>
</template>

<script setup lang="ts">
function sleep(time: number) {
  return new Promise<void>((resolve, reject) => {
    setTimeout(() => {
      resolve();
    }, time);
  });
}
await sleep(3000);
```

## Implementing v-model with Custom Components.

```
const props = defineProps<{
  visible: boolean;
}>();

const visible = computed({
  set: (val) => emit('update:visible', val),
  get: () => props.visible,
});

<a-modal v-model:visible="visible" centered :footer="null">
```

## Custom Modifiers — modelModifiers

In addition to the few set by the officials, modifiers can also be implemented by oneself.

For example, in the next example, different logical judgments can be made based on whether ‘capitalize’ exists in props.modelModifiers.

```
<!-- Parent -->
<MyComponent v-model.capitalize="myText" />

<!-- Child -->
const props = defineProps({
  modelValue: String,
  modelModifiers: { default: () => ({}) }
})

defineEmits(['update:modelValue'])

console.log(props.modelModifiers) // { capitalize: true }
<template>
  <input
    type="text"
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
  />
</template>
```

or

```
interface IProps {
  // selectPlan: 'Yearly' | ' Monthly';
  selectPlan: string;
  modelModifiers?: { default: () => Record<string, unknown> };
}
const props = withDefaults(defineProps<IProps>(), {
  selectPlan: 'Yearly',
});
console.log(props.modelModifiers); // {aaa:true}

<Paypal v-model.aaa="xxxx" ></Paypal>
```

## Attribute Penetration

“Pass-through attributes” refer to attributes or v-on event listeners that are passed to a component but are not declared as props or emits by the component. The most common examples are class, style, and id.

When a component is rendered as a single element root, the passed-through attributes are automatically added to the root element. For example, if we have a <MyButton> component, its template looks like this:

```
<!-- <MyButton> temp-->
<button class="aa">click me</button>

<!-- use -->
<MyButton class="large" />
```

At this time, when this button is truly rendered, the class attribute of this node node is 2, “aa” and “large”.

**Disabling Attribute Inheritance**

From version 3.3, you can also use defineOptions directly in `<script setup>`

The same rules also apply to v-on event listeners

```
<script setup>
defineOptions({
  inheritAttrs: false
})
</script>
```

The click listener will be added to the root element of `<MyButton>`, that is, on the native `<button>` element. When the native `<button>` is clicked, it will trigger the parent component’s onClick method. Likewise, if the native button element itself also binds an event listener through v-on, both this listener and the listener inherited from the parent component will be triggered.

**Accessing Pass-through Attributes in JavaScript**

```
<script setup>
import { useAttrs } from 'vue'

const attrs = useAttrs()
</script>
```

## Principles of Using Provide Injection

When providing/injecting reactive data, it is recommended to keep any changes to the reactive state as much as possible within the provider component. This can ensure that the declaration and modification operations of the provided state are all contained within the same component, making it easier to maintain.

Sometimes, we might need to change the data in the injection component. In this case, we recommend declaring and providing a method function to change the data within the provider component.

```
<script setup>
import { provide, ref } from 'vue'

const location = ref('North Pole')

function updateLocation() {
location.value = 'South Pole'
}

provide('location', {
location,
updateLocation
})
</script>
```

**Using Symbol as Injection Name**

If you are building a large application with many dependency providers, or you are writing a component library for other developers, it is best to use Symbols as injection names to avoid potential conflicts.

```
// keys.js
export const myInjectionKey = Symbol()

js
// provide Comp
import { provide } from 'vue'
import { myInjectionKey } from './keys.js'

provide(myInjectionKey, { /*
  data
*/ });

js
// inject Comp
import { inject } from 'vue'
import { myInjectionKey } from './keys.js'

const injected = inject(myInjectionKey)
```

## Compilation Build Steps and Component Templates

When you use Vue, there are two main ways to handle component templates: one is the method with no build steps, the other is with build steps.

- No build step method: In the instance of no build steps, you can directly write the component template in the page’s HTML code or as an inline JavaScript string. However, in this case, for the dynamic template to work properly, Vue needs to run the template compiler in the browser. This means that Vue will dynamically compile the template in the browser, then execute the rendering.

- Use of build steps: When you use build steps, the template is pre-compiled, without the need to run the template compiler in the browser. This can reduce the size of the client code and improve performance. To adapt to different optimization needs, Vue provides several formats of “build files”.

- Files with the prefix vue.runtime.* are versions that only include the runtime, and do not include the compiler. If you use this version, all templates must be pre-compiled in the build step.

- Files not containing .runtime are the full version, including the compiler, and can compile templates directly in the browser. However, as it includes the compiler, the file size will increase by about 14kb.

By default, the toolchain will use the version that only contains the runtime, because in the case of using build steps, all the templates in single file components (SFC) have been precompiled. However, if you still need the browser’s template compiler in the case of build steps, you can change the configuration of the build tool and change the version of Vue to the corresponding version, such as vue/dist/vue.esm-bundler.js.

In summary, this passage explains how to handle Vue component templates in different circumstances, as well as how to select the appropriate version of Vue according to whether or not build steps are used, in order to optimize the size and performance of client-side code.

## Method Handlers and Inline Handlers in Vue3

What’s the difference between calling a method via `@click=”foo()”` and directly using `@click=”foo”`?

`foo()` will eventually turn into a function like `() => foo()`, which means that foo cannot get the event argument. But for direct foo, foo can receive the event argument.

## The generic attribute on the component `<script>` tag declares generic type parameters

When writing subcomponents, if you are unsure of the type of the argument passed in from the outside, you can use the generic attribute to declare a generic.

In this way, when the outside world uses the subcomponent, the type of data it passes in is certain. The subcomponent gets this type and throws it back, so you can get the type when you use scoped slots.

```
// Child
<template>
  <div>
    list
    <slot name="header" :expos="props.list"></slot>
  </div>
</template>

<script setup lang="ts" generic="T">
const props = defineProps<{
  list: T[];
}>();
</script>
```

## Type Annotations for Component Template References

Sometimes, you may need to add a template reference to a child component in order to call its public methods. For example, we have a MyModal child component, which has a method to open the modal box:

```
<!-- MyModal.vue -->
<script setup lang="ts">
import { ref } from 'vue'

const isContentShown = ref(false)
const open = () => (isContentShown.value = true)

defineExpose({
  open
})
</script>
```

To get the type of MyModal, we first need to use typeof to get its type, then use the TypeScript built-in tool type InstanceType to get its instance type.

```
<!-- App.vue -->
<script setup lang="ts">
import MyModal from './MyModal.vue'

const modal = ref<InstanceType<typeof MyModal> | null>(null)

const openModal = () => {
  modal.value?.open()
}
</script>
```

## Advantages of Composition API

1. **Better Logic Reuse**: The most fundamental advantage of the Composition API is that it allows us to achieve more concise and efficient logic reuse by combining functions. This is a significant improvement over the Options API, which primarily employs mixins for logic reuse.

2. **Enhanced Code Organization**: The Composition API allows for more flexible code organization. In the Options API, the codes for different functional operations might be scattered throughout various sections of a file. The Composition API, on the other hand, facilitates the organization of codes such that they can be focused in the same area.

3. **Improved Type Inference**: The composition API brings about better type inference in Integrated Development Environments (IDE), thereby enhancing the developer’s coding experience with better autocompletion, less ambiguity, and fewer errors.

4. **Smaller Production Bundle**: The use of Composition API leads to smaller production bundles. This happens because the Composition API’s template code is compiled into inline functions that share the same scope with the script setup code, hence the compiled template can directly reference local variables. This characteristic of the Composition API also makes it more compression-friendly.

Using Composition API with `<script setup>` is more efficient than the equivalent option API, and it is also more friendly to code compression. This is because the component template written in `<script setup>` form is compiled into an inline function, and the code in `<script setup>` is in the same scope. Unlike the Options API which needs to rely on the `this` context object to access properties, the compiled template can directly access variables defined in `<script setup>`, without the need for proxying from the instance. This is more friendly to code compression because local variable names can be compressed, but object property names cannot.

