---
title: 'Vue3 develop document'
date: 2024-01-21T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/0*WzmFm3IVFVQLnzcA"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

## Get this

In Vue2, `this` in each component points to the current component instance, and `this` also contains globally mounted things, routes, state management, etc.

However, there is no `this` in the Composition API of Vue3. If you want a similar usage, there are two, one is to access the current component instance, and the other is to access the global instance. You can print it out by yourself and take a look.

```
<script setup>
import { getCurrentInstance } from 'vue'

// Proxy is the current component instance, which can be understood as this at the component level, without the global instance, route, state management, etc.
const { proxy, appContext } = getCurrentInstance()

// This global is the global instance.
const global = appContext.config.globalProperties
</script>
```

## Global Registration(Property/Function)

In Vue2, if we want to mount something globally, we usually do it as follows, and then we can get it through `this.xxx` in all components.

```
Vue.prototype.xxx = xxx
```

But in Vue3 you can’t write like this. Instead, it changes to a global object that can be accessed by all components, which is the object of the global instance mentioned above. For example, make global registration in main.js.

```
// main.js
import { createApp } from 'vue'
import App from './App.vue'
const app = createApp(App)
// add global property
app.config.globalProperties.name = 'xxx'
```

Call it in other components.

```
<script setup>
import { getCurrentInstance } from 'vue'
const { appContext } = getCurrentInstance()

const global = appContext.config.globalProperties
console.log(global.name)
</script>
```

## Template

In Vue2, there can only be one root node, while Vue3 supports multiple root nodes, which everyone knows.

In essence, each component in Vue3 is still a root node, because the DOM tree can only be a tree structure. It’s just that Vue3 added a judgment during the compilation stage. If the current component has more than one root element, it will add a `fragment` component to package this multi-root component, which means this component still only has one root node. And the `fragment`, like `keep-alive`, is an built-in component that will not be rendered.

## Get DOM

```
<template>
    <el-form ref="formRef"></el-form>
    <child-component />
</template>
<script setup lang="ts">
import ChildComponent from './child.vue'
import { getCurrentInstance } from 'vue'
import { ElForm } from 'element-plus'

// Method one, the variable name must be the same as the ref attribute on the DOM, which can automatically form a binding.
const formRef = ref(null)
console.log(formRef.value)

// Method two
const { proxy } = getCurrentInstance()
proxy.$refs.formRef.validate((valid) => { ... })

// Method three，For example, in ts, you can directly get the component type.
// You can get sub-components like this.
const formRef = ref<InstanceType<typeof ChildComponent>>()
// You can also get the component type of element ui like this.
const formRef = ref<InstanceType<typeof ElForm>>()
formRef.value?.validate((valid) => { ... })
</script>
```

## Initialization

In Vue2, when you enter the page and then request the interface, or some other initial operations, they are generally placed in `created` or `mounted`, while in Vue3 `beforeCreated` and `created` are not used, because `setup` is executed before these two, and having these two would be superfluous.

So the content that was used in the `beforeCreated / created / beforeMount / mounted` hooks can be directly placed in the setup in Vue3, or in `onMounted/onBeforeMount`.

```
<script setup>
import { onMounted } from 'vue'

// Request function.
const getData = () => {
    xxxApi.then(() => { ... })
}

onMounted(() => {
    getData()
})
</script>
```

## Unbinding

In Vue2, there are generally two methods for operations such as clearing timers, listeners, etc:

One is to use `$once` in conjunction with hook: `BeforeDestroy`, which is not supported by Vue3.
The other is to use the `beforeDestroy` / `deactivated` hooks. In Vue3, the hook functions are just renamed.

```
<script setup>
import { onBeforeUnmount, onDeactivated } from 'vue'

// Before the component is unloaded, it corresponds to beforeDestroy in Vue2.
onBeforeUnmount(() => {
    clearTimeout(timer)
    window.removeAddEventListener('...')
})

// Exit cache component, corresponds to deactivated in Vue2.
onDeactivated(() => {
    clearTimeout(timer)
    window.removeAddEventListener('...')
})
</script>
```

## Ref and Reactive

Both are used to create responsive objects, `ref` is commonly used to create basic types, `reactive` is typically used to create responsiveness, this is advocated by official, yet not necessarily in reality. Some people use `ref` to define arrays, some people define only one `reactive` in a component and put all data in it, just like Vue2’s data, and some people use both.

There are two points to note:

If `ref` is passed in an referenced type, the internal source code also calls reactive to implement it.

The property returned by ref is used directly in the template, but in JS it needs to be obtained through `.value`, as shown below. Because the ref returns a wrapped object.

```
<template>
    <div>{{ count }}</div>
</template>
<script setup>
import { ref, reactive } from 'vue'
const count = ref(1)

const arr = ref([])
console.log(arr.value) // []

const data = reactive({
    name: 'xxx',
    age: 18,
    ...
})
console.log(data.name)

</script>
```

Why does `ref` have to return a wrapped object? It’s well known that data in Vue2 all return an object.

Because object reference types can be used for proxies or hijacking, if only the basic type is returned, it is stored in the stack, and once execution in the execution stack is completed, it is recovered, with no possibility of adding a proxy or hijacking. Naturally it’s impossible to track subsequent changes, so it has to return an object, so that it can be responsive.

## toRef and toRefs

The common point of these two methods is to create responsive references, mainly used to take the properties out of the responsive objects, or destructure the responsive objects, and the destructured property values are still responsive. If you destructure them directly without these two methods, you will lose the responsive effect.

The main advantage is that we can use the direct variable xxx without needing `data.xxx`. And when we modify xxx, we are also directly modifying the underlying object’s properties.

The difference between these two: The one with “s” and without “s”, are singular and plural. The meaning is to take one or to take many.

```
<script setup>
import { reactive, toRef, toRefs } from 'vue'

const data = reactive({
    name: 'xxx',
    age: 18
})

// Although you can get name/age this way, they become ordinary variables and lose reactive effect.
const { name, age } = data

// A responsive property is taken out.
const name = toRef(data, 'name')

// The properties that are destructured in this way all have responsive effects.
const { name, age } = toRefs(data)

// Whether it's toRef or toRefs, modifying in this way will change the name in data.
// That means changing the properties of the source object, which is the typical behavior of reactive.
name.value = 'xxx'
</script>
```

## watch

`watch` is used to monitor an existing property, and to do certain operations when changes occur. The following three ways of writing are commonly used in Vue2.

```
watch: {
    userId: 'getData',
    userName (newName, oldName) {
        this.getData()
    },
    userInfo: {
        handler (newVal, newVal) { this.getData() },
        immediate: true,
        deep: true
    }
}
```

In Vue3, the listening syntax is much more enriched.

Vue3’s `watch` is a function that can receive three parameters, the first parameter is the property to be monitored, the second one is the callback function to receive the new and old values, and the third one is the configuration item.

```
<script setup>
import { watch, ref, reactive } from 'vue'

const name = ref('xxx')
const data = reactive({
    age: 18,
    money: 100000000000000000000,
    children: []
})

// Listening ref attribute
watch(name, (newName, oldName) => { ... })

// This is how you listen for other properties, routes, or state management.
watch(
    () => data.age, 
    (newAge, oldAge) => { ... }
)

// To listen to multiple properties, put multiple values ​​in the array, and the returned new and old values ​​are also in the form of an array.
watch([data.age, data.money], ([newAge, newMoney], [oldAge, oldMoney]) => { ... })

// The third parameter is an object, which is a configurable item with five configurable properties.
watch(data.children, (newList, oldList) => { ... }, {
    // Similar to Vue2
    immediate: true,
    deep: true,
    // The execution timing of the callback function, which is called by default before the component is updated. If it's called after the update, change it to 'post'.
    flush: 'pre', // The default value is 'pre', you can change it to 'post' or 'sync'.
    // use debug
    onTrack (e) { debugger }
    onTrigger (e) { debugger }
})
</script>
```

Inside the watch callback function, you can accept the third parameter onInvalidate, `which` is a function that clears side effects. For the first time, the callback function of the listening (handler) is not going to trigger `onInvalidate`, and subsequently, will trigger `onInvalidate` by default each time.

In other words, it’s default operation mechanism called before updating, for example, in the following code, when the key triggers an update, it will print 222 first and then print xxx. If you need to call after updating, you can add flush: post in the third configuration of watch.

```
// The callback function receives a parameter, which is a function used to clear side effects.
watch(key, (newKey, oldKey, onInvalidate) => {
    console.log('xxx')
    // By default, the DOM obtained is the DOM before the update. If it is flush: post, you can get the DOM after the update.
    console.log('DOM：', dom.innterHTML)
    onInvalidate(() => {
        console.log(2222)
    })
})
```

The use scenario of `onInvalidate` is such as: for example, there are some asynchronous operations in the callback function (handler) of listening, and when triggering `watch` again, it can be used to cancel/ignore/reset/initialize some operations of the previous unfinished asynchronous tasks, such as canceling the unfinished request when triggering `watch` last time.

## watchEffect

In Vue3, in addition to `watch`, there is also a `watchEffect`. The differences are:

`watch` is to listen to one or more values passed in, and it will return new and old values when triggered, and by default it will not execute for the first time.

`watchEffect` is a function that is executed immediately, so it will be executed by default for the first time, and there is no need to pass in listening content. It will automatically collect the data sources in the function as dependencies, and it will re-execute the function when the dependencies change. (a bit like computed), and it will not return new and old values.

The timing of side effect clearance and refreshment of side effects are the same, the difference is that in `watch`, it will be passed as the third argument of the callback, in `watchEffect` it is the first argument of the callback function.

Normally, both will stop listening automatically after the component is `destroyed/unmounted`, but there are exceptions, such as asynchronous methods, listeners created in setTimeout need to manually stop listening, and the stop method is as follows.

```
// Assigning a listening method.
const unwatch = watch('key', callback)
const unwatchEffect = watchEffect(() => {})
// When you need to stop listening, manually call to stop listening.
unwatch()

unwatchEffect()
```

Use of `watchEffect`:

```
<script setup>
import { watchEffect } from 'vue'

// Normal use.
watchEffect(() => {
    // It will automatically collect the properties used by this function as dependencies for monitoring.
    // It monitors the userInfo.name property, and will not monitor userInfo.
    console.log(userInfo.name)
})

// There are two parameters. The first one is to trigger the monitoring callback function, and the second one is optional configuration items.
watchEffect(() => {...}, {
    // This is where the configuration items are. The meaning is the same as watch, but there are only three available configuration items.
    flush: 'pre',
    onTrack (e) { debugger }
    onTrigger (e) { debugger }
})

// The callback function receives one parameter, which is the function to clear side effects, similar to the one in watch.
watchEffect(onInvalidate => {
    console.log('xxx')
    onInvalidate(() => {
        console.log(2222)
    })
})
</script>
```

If you need to change the configuration item flush to post or sync in `watchEffect`, you can directly use the alias, as follows.

```
watchEffect(() => {...}, {
    flush: 'post',
})
//  it's the same as the one below
watchPostEffect(() => {})
-----------------------------
watchEffect(() => {...}, {
    flush: 'sync',
})
//  it's the same as the one below
watchSyncEffect(() => {})
```

## computed

In Vue2, the most common use cases for computed are: `mapGetters/mapState` for fetching property from state management, getting property from a URL, conditional judgments, type conversions, etc., and it supports both function and object writing methods.

In Vue3, `computed` is no longer an object, but a function. The usage is somewhat similar in general, the first argument of the function is the listener source, used to return the `computed` new value. It also supports object writing methods, and the second argument can be used for debugging.

```
<script setup>
import { computed } from 'vue'
const props = defineProps(['visible', 'type'])
const emit = defineEmits(["myClick"])

// Function method, computed type.
const isFirst = computed(() => props.type === 1)

// Object method
const status = computed({
    get () { return props.visible }, // It is equivalent to this.visible in Vue2.
    set (val) { emit('myClick', val) } // It is equivalent tothis.$emit('input', val)in Vue2.
})

// The second argument of computed is also an object, used for debugging.
const hehe = computed(Parameter one either of the above can be used， {
    onTrack (e) { debugger }
    onTrigger (e) { debugger }
})
</script>
```

## nextTick

The usage method of `nextTick`, excluding the use of this, everything else is exactly the same as Vue2, and there are still three ways.

```
<script setup>
import { nextTick} from 'vue'

// one
const handleClick = async () => {
  await nextTick()
  console.log('xxx')
}

// two
nextTick(() => {
    console.log('xxx')
})

// three
nextTick().then(() => {
    console.log('xxx')
  })
</script>
```

## mixins and hooks

In Vue2, logic extraction and reuse generally use `mixins`, but there are three disadvantages:

- There is no independent namespace, `mixins` will cause naming conflicts with the internals of the component

- If you don’t go through the code, you won’t know what’s in the imported `mixins`

- When multiple `mixins` are introduced, you don’t know which one the mixin you are using comes from

The hooks syntax for logic extraction and reuse in Vue3 is actually just a function that can accept arguments and use the returned values. Or, it can be understood this way: how to write commonly used methods that need to be encapsulated? You can do it just like that in Vue3.

```
// xxx.js
expport const getData = () => {}
export default function unInstance () {
    ...
    return {...}
}

// xxx.vue
import unInstance, { getData } from 'xx.js'
const { ... } = unInstance()
onMounted(() => {
    getData()
})
```

Regarding how to write more elegant code with hooks, one needs to write more and practice more. It’s not something that can be mastered with a few sentences and lines of code.

## Component communication

There are several ways to communicate between components in Vue3:

- props + defineProps

- defineEmits

- defineExpose / ref

- useAttrs

- v-model (supports multiple)

- provide / inject

- Vuex / Pinia

## multiple v-model

In Vue2, only one `v-model` can be written on each component. If the child component does not write a model, it can be received by props by default, and modifications are made through the `this.$emit(‘input’)` event.

In Vue3, each component supports writing multiple `v-model`, eliminating the need for `.sync` and model renaming operations. When writing `v-model`, you need to include the name, as follows:

```
// parent
<template>
    <child v-model:name="name" v-model:age="age" />
</template>
<script setup>
import { ref } from "vue"
const name = ref('xxx')
const age = ref(18)
</script>

// child
<script setup>
const emit = defineEmits(['update:name', 'update:age'])

const handleClick = () => {
    console.log('clicked')
    emit('update:name', 'yyy')
}
</script>
```

## State Management

The usage of Vuex is basically the same as Vue2. If you are starting from scratch, it’s recommended to use Pinia directly.

## router

```
// main.js
import { createApp } from 'vue'
import App from './App.vue'
import Router from './router'
const app = createApp(App)
app.use(Router)
...

// router/index.js
import { createRouter, createWebHistory, RouteRecordRaw } from 'vue-router'
const routes = [
  { path: '/', redirect: { name: 'login' } }
]
const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes
})
export default router

// .vue file
<script setup>
import { useRoute, useRouter } from "vue-router"
// 'route' corresponds to this.$route in Vue2.
const route = useRoute()
// router corresponds to this.$routerin Vue2.
const router = useRouter()
</script>
```

## CSS Style Penetration

In Vue2, when it’s not possible to modify the style of sub-components or components within a library in `scoped`, you can use CSS style penetration. Regardless of whether it’s Less or SASS, you use `/deep/ .class {}` for style penetration. However, Vue3 does not support the `/deep/` syntax, it has been replaced with `:deep(.class)` for style penetration.

```
<style lang="scss" scoped>
// If this doesn't work
.el-form {
    .el-form-item { ... }
}
// Vue2
/deep/ .el-form {
    .el-form-item { ... }
}
// Vue3
:deep(.el-form) {
    .el-form-item { ... }
}
</style>
```

## Binding JS Variables with CSS

```
<template>
    <div class="name">xxx</div>
</template>
<script setup>
import { ref } from "vue"
const str = ref('#f00') // red
</script>
<style scoped lang="scss">
.name {
    background-color: v-bind(str);
}
</style>
```
