---
title: 'Understanding Suspense in Vue3'
date: 2024-03-21T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/0*J3V7D8PQIiGwlztx.jpeg"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: "Vue 3 provides a built-in solution for asynchronous tasks called Suspense, designed to enhance user experience when interacting with..."
---

## Overview

Vue 3 provides a built-in solution for asynchronous tasks called Suspense, designed to enhance user experience when interacting with parts of an app that depend on asynchronous data. Suspense allows you to show alternative content, such as loading placeholders, while waiting for an asynchronous action to complete, like data fetching or a dynamic component’s resolution. Once the action is resolved, Suspense will automatically transition to the actual content, thus eliminating unsightly blank states or spinners and resulting in a smoother user experience.

Suspense is particularly useful in two main scenarios:

- Asynchronous Component Loading: Suspense helps when your application needs to wait for a component to finish loading asynchronously before it can be rendered.

- Data Fetching: When components need to wait for asynchronous data to be fetched before they render to avoid showing an empty state or a loading indicator, Suspense streamlines this experience.

---

## Usage

To utilize Suspense, you need to grasp two fundamental concepts: the `<Suspense>` component and the v-slot directive.

1. `<Suspense>`

The `<Suspense>` component is a vital part of Vue 3. It wraps around content that may involve asynchronous loading. When the content within `<Suspense>` embarks on asynchronous actions, it displays a fallback (placeholder content) until the asynchronous operation is completed.

```
<template>
  <Suspense>
    <template #default>
      <!-- Content that loads asynchronously -->
      <AsyncComponent />
    </template>
    <template #fallback>
      <!-- Placeholder content -->
      <LoadingIndicator />
    </template>
  </Suspense>
</template>
```

In the example above, `<Suspense>` wraps an asynchronously loaded component `<AsyncComponent />`, while providing a placeholder content `<LoadingIndicator />` to display during the execution of the asynchronous operation.

2. `v-slot` Directive

The `v-slot` directive is used to define the content for fallback and default slots. In the example shown earlier, we use #default and #fallback to specify the content for the two slots.

### Asynchronous Component Loading

Suspense is most commonly used to handle the loading of asynchronous components. Vue 3 enables you to load components on-demand, reducing your application's initial loading time. The example below demonstrates how to use Suspense to manage the loading of asynchronous components:

```
<template>
  <Suspense>
    <template #default>
      <AsyncComponent />
    </template>
    <template #fallback>
      <LoadingIndicator />
    </template>
  </Suspense>
</template>
<script>
import { defineAsyncComponent } from 'vue';
const AsyncComponent = defineAsyncComponent(() =>
  import('./AsyncComponent.vue')
);
export default {
  components: {
    AsyncComponent
  }
};
</script>
```

```
<template>
  <h1>this is async component</h1>
</template>

<script>
export default {
  name: "AsyncComponent",
  async setup() {
    await sleep(3000); //Simulated data request
  }
};
</script>
```

In the example shown above, AsyncComponent is a loaded on-demand, wrapped with the defineAsyncComponent function. While AsyncComponent is loading asynchronously, Suspense displays the LoadingIndicator until the component has finished loading.

Vue3 Suspense allows us to load multiple asynchronous components in parallel as opposed to sequentially loading one by one as in previous versions. This can reduce waiting time, improve loading speed, and enhance user experience.

### Data Fetching

Suspense can also be applied to manage asynchronous data fetching. This is particularly beneficial when you want to wait for data to finish loading before rendering components to prevent displaying a blank space or a loading indicator.

```
<template>
  <Suspense>
    <template #default>
      <UserData :user-id="userId" />
    </template>
    <template #fallback>
      <LoadingIndicator />
    </template>
  </Suspense>
</template>
```

In the example provided above, the UserData component takes a userId prop and loads user data based on that ID. While the data is loading, it will display user information, but during the loading phase, Suspense will show the LoadingIndicator.

### Used with routing

Working with routing systems is especially convenient. When the components have not yet finished loading, do not display the router-view but rather render the content of the fallback, thereby globally adding a loading process:

```
<router-link to="/home">Home</router-link>|
<router-link to="/about">About</router-link>
<Suspense>
  <template #default>
    <router-view />
  </template>
  <template #fallback>
    <div class="loading"></div>
  </template>
</Suspense>
```

In this configuration, instead of immediately presenting the `router-view`, Suspense renders a loading indicator during the loading phase until the associated components are fully loaded.

### Error handling

Suspense also offers a mechanism for handling the failure of asynchronous operations. You can use try/catch inside the asynchronous operation to capture exceptions, and then use the $suspend method within the catch block to notify Suspense. This will trigger the display of the fallback.

```
<template>
  <Suspense>
    <template #default>
      <AsyncDataComponent />
    </template>
    <template #fallback>
      <ErrorIndicator />
    </template>
  </Suspense>
</template>
<script>
import { ref, defineAsyncComponent, onErrorCaptured } from 'vue';

export default {
  setup() {
    const error = ref(null);
    onErrorCaptured(e => {
      error.value = e;
      return false;
    });
    return { error };
  },
  components: {
    AsyncComponent: defineAsyncComponent(() => {
      return import("../components/async-component.vue");
    })
  },
}
</script>
```

```
<script>
export default {
  name: "AsyncComponent",
  async setup() {
    if (Math.random() > 0.5) {
      await sleep(1000);
    } else {
      throw new Error("error message");
    }
  }
};
</script>
```

In the example above, AsyncDataComponent captures the error using try/catch and passes the error to Suspense by throwing it, thus triggering the display of the fallback.

### Suspense Hooks

Vue Suspense also provides several lifecycle hooks that allow you to execute code at various stages of asynchronous operations. These hooks include `onPending`, `onResolve`, and `onReject`, which can be used to perform additional logic such as logging, performance analysis, or error handling.

```
<template>
  <Suspense
      :onPending="onPending"
      :onResolve="onResolve"
      :onReject="onReject"
  >
    <template #default>
      <AsyncComponent />
    </template>
    <template #fallback>
      <LoadingIndicator />
    </template>
  </Suspense>
</template>
<script>
import { defineAsyncComponent } from 'vue';
const AsyncComponent = defineAsyncComponent(() =>
  import('./AsyncComponent.vue')
);
export default {
  components: {
    AsyncComponent
  },
  methods: {
    onPending() {
      console.log("loading...");
    },
    onResolve() {
      console.log("complete。");
    },
    onReject(error) {
      console.error("fail：", error);
    },
  },
};
</script>
```
