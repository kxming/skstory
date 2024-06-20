---
title: 'Understanding effectScope in Vue3'
date: 2024-06-17T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Mejgw2yY0uDgYRxE8Vo1yA.png"
categories: ["Code","Javascript"]
tags: ["Code","Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "In Vue’s component setup(), effects will be collected and bound to the current instance."
---

## Overview

In Vue’s component `setup()`, effects will be collected and bound to the current instance. When the instance get unmounted, effects will be disposed automatically. This is a convenient and intuitive feature. However, when we are using them outside of components or as a standalone package, it’s not that simple.

Especially when we have some long and complex composable code, it’s laborious to manually collect all the effects.

It’s also easy to forget collecting them (or you don’t have access to effects created in the composable functions) which might result in memory leakage and unexpected behavior.

`effectScope` is introduced to solve this problem, and this feature was added in vue3.2, which belongs to the advanced content of the reactive system. This is the [official documentation](https://vuejs.org/api/reactivity-advanced.html#effectscope) explanation：

> Creates an effect scope object which can capture the reactive effects (i.e. computed and watchers) created within it so that these effects can be disposed together. For detailed use cases of this API, please consult its corresponding [RFC](https://github.com/vuejs/rfcs/blob/master/active-rfcs/0041-reactivity-effect-scope.md).

Literally, it is the effect scope, used to collect the side effects created within it and to handle them uniformly. The main content of the API is to create an effect scope using `effectScope`, which can capture the reactive side effects (i.e., `computed` properties and `watchers`) created within it, so that the captured side effects can be dealt with together. Use `getCurrentScope` to return the current active effect scope. Use `onScopeDispose` to register a handler callback on the current active effect scope. This callback will be called when the related effect scope stops.

## Usage

`effectScope` is mainly used in hooks, and `effectScope` is extensively utilized in [vueuse](https://vueuse.org/guide/).

### Basic

```
// effect, computed, watch, watchEffect created inside the scope will be collected

// Creating a new scope
const scope = effectScope()

// a scope allows the execution of a run function (which accepts a function as an argument and returns the return value of that function) and captures all the effects created during the execution of that function, including APIs that can create effects, such as computed, watch, and watchEffect
scope.run(() => {
  const doubled = computed(() => counter.value * 2)

  watch(doubled, () => console.log(doubled.value))

  watchEffect(() => console.log('Count: ', doubled.value))
})

// to dispose all effects in the scope
// When calling scope.stop(), all captured effects will be cancelled, including nested Scopes, which will also be recursively cancelled.
scope.stop()
```

### Nested Scopes

Nested scopes should also be collected by their parent scope. And when the parent scope gets disposed, all its descendant scopes will also be stopped.

```
const scope = effectScope()

scope.run(() => {
  const doubled = computed(() => counter.value * 2)

  // not need to get the stop handler, it will be collected by the outer scope
  effectScope().run(() => {
    watch(doubled, () => console.log(doubled.value))
  })

  watchEffect(() => console.log('Count: ', doubled.value))
})

// dispose all effects, including those in the nested scopes
scope.stop()
```

### Detached Nested Scopes

`effectScope` accepts an argument to be created in "detached" mode. A detached scope will not be collected by its parent scope.

This also makes usages like **lazy initialization** possible.

```
let nestedScope

const parentScope = effectScope()

parentScope.run(() => {
  const doubled = computed(() => counter.value * 2)

  // with the detected flag,
  // the scope will not be collected and disposed by the outer scope
  nestedScope = effectScope(true /* detached */)
  nestedScope.run(() => {
    watch(doubled, () => console.log(doubled.value))
  })

  watchEffect(() => console.log('Count: ', doubled.value))
})

// disposes all effects, but not `nestedScope`
parentScope.stop()

// stop the nested scope only when appropriate
nestedScope.stop()
```

`onScopeDispose`

The global hook `onScopeDispose()` serves a similar functionality to `onUnmounted()`, but works for the current scope instead of the component instance. This could benefit composable functions to clean up their side effects along with its scope. Since `setup()` also creates a scope for the component, it will be equivalent to `onUnmounted()` when there is no explicit effect scope created.

```
import { onScopeDispose } from 'vue'

const scope = effectScope()

scope.run(() => {
  onScopeDispose(() => {
    console.log('cleaned!')
  })
})

scope.stop() // logs 'cleaned!'
```

### Getting the Current Scope

```
import { getCurrentScope } from 'vue'

getCurrentScope() // EffectScope | undefined
```

## Examples

Some composables setup global side effects. For example the following `useMouse()` function:

```
function useMouse() {
  const x = ref(0)
  const y = ref(0)

  function handler(e) {
    x.value = e.x
    y.value = e.y
  }

  window.addEventListener('mousemove', handler)

  onUnmounted(() => {
    window.removeEventListener('mousemove', handler)
  })

  return { x, y }
}
```

If the `useMouse` hook to obtain coordinates is used by multiple components, it will add multiple listeners, increasing performance cost. In this case, using `effectscope` is more performance-friendly.

```
function useMouse() {
  const x = ref(0)
  const y = ref(0)

  function handler(e) {
    x.value = e.x
    y.value = e.y
  }

  window.addEventListener('mousemove', handler)

  onScopeDispose(() => {
    window.removeEventListener('mousemove', handler)
  })

  return { x, y }
}

function createSharedComposable(composable) {
  let subscribers = 0
  let state, scope

  const dispose = () => {
    if (scope && --subscribers <= 0) {
      scope.stop()
      state = scope = null
    }
  }

  return (...args) => {
    subscribers++
    if (!state) {
      scope = effectScope(true)
      state = scope.run(() => composable(...args))
    }
    onScopeDispose(dispose)
    return state
  }
}

const useSharedMouse = createSharedComposable(useMouse)

export default useSharedMouse
```
