---
title: 'Understanding and Practice of the Vue3 Composition API'
date: 2024-02-14T16:22:33+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*SirXXHZL0xDpmuI4y36vSw.jpeg"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: "Composition API brings a new way to write component logic, and provides a more flexible, composable, and reusable code structure..."
---

## Introduction

The upgrade of Vue.js to Vue3 brings many exciting features and improvements, among which the most notable is the introduction of Composition API. It brings a new way to write component logic, and provides a more flexible, composable, and reusable code structure, enabling developers to better organize and manage complex front-end logic.

Using the Composition API can avoid writing a large amount of code in the Options API’s large object that is difficult to understand when writing complex components. Therefore, it is necessary to delve into Vue 3’s Composition API, use it proficiently in projects, and write high-quality maintainable code.

---

## Composition API Introduction

### Options API Review

In Vue 2, we primarily use the Options API to create and manage Vue components. The main idea of the Options API is to define different parts of a component (such as data, methods, computed, etc.) in different options. The advantage of this approach is that it is clear in structure, easy to start with, and convenient for editing small components. However, as the component becomes more and more complex, this approach may lead to a decrease in code readability and maintainability.

For example, suppose we have a very complex component that involves multiple functional modules. In the Options API, we need to scatter the code of these functional modules into different options. This may confuse us in reading and understanding the code, because we need to jump back and forth between different options.

The Options API has some other issues as well. For example, it does not support type inference, which makes using Vue in TypeScript difficult. Moreover, the Options API does not support code reuse. Although we can use mixins to reuse code, mixins have their own problems, such as naming conflicts and unclear data sources, etc.

```
export default {
  data() {
    return {
      count: 0,
    };
  },
  computed: {
    doubleCount() {
      return this.count * 2;
    },
  },
  methods: {
    increment() {
      this.count++;
    },
  },
};
```

### Composition API Addresses Pain Points

To solve the issues with the Options API, Vue 3 introduced the Composition API. The main idea of the Composition API is to provide a new and more flexible way to organize and reuse code. With the Composition API, we can organize code by functional modules rather than Vue options.

For example, suppose we have a very complex component that involves multiple functional modules. In the Composition API, we can put the code for each functional module together, rather than scattered into different options. This makes it easier for us to understand and maintain complex components.

In addition, the Composition API also provides better type inference, making it easier for us to use Vue in TypeScript. All in all, using the Composition API can make our code cleaner, more readable, and maintainable.

```
import { ref, computed } from 'vue';

export default {
  setup() {
    const count = ref(0);
    const doubleCount = computed(() => count.value * 2);

    function increment() {
      count.value++;
    }

    return {
      count,
      doubleCount,
      increment,
    };
  },
};
```

---

## Composition API Core Concepts

### Setup Function

In the Composition API, we primarily write code in the setup function. The setup function is a special function that is called when the component is initialized, and we can define and return our reactive data and functions inside this function.

The setup function receives two parameters: `props` and `context`

1. props: is an object, a way of component communication, and cannot use ES6 destructuring, it will eliminate the reactivity of the prop, at this time you need to use toRef or toRefs to take the value, using it has another advantage, can follow the single-way data flow of props, and modify props value

2. context: context object, attributes are `attrs`, `slots`, `emit`, `expose`

  - attrs is a non-reactive object, mainly receiving no-props attributes, often used to pass some style attributes

  - slots is an object containing all slots, where slots.default() gets an array of slot contents

  - emit replaces the previous this.$emit due to the lack of this in the setup, used for child-to-parent transmission, triggering custom events

  - expose controls the exposed properties and methods of the component, no interface is exposed by default in the script setup

```
setup(props,context){
   const { msg，ans } = toRefs(props)
   console.log(msg.value);
   console.log(ans.value);

  const { attrs, slots, emit, expose } = context
    // attrs gets the attribute value passed by the component,
    // slots inside the component's slots
    // emit custom event for child components
    // expose the exposed interface
  }
```

The setup function is executed before created, and there is no this inside, so this related stuff cannot be mounted. We can use Vue’s various reactive APIs, such as reactive and ref. The properties and methods within the setup that want to be used in the component template must be returned and exposed

```
import { ref } from 'vue';

export default {
  setup() {
    const count = ref(0);

    function increment() {
      count.value++;
    }

    return {
      count,
      increment
    };
  }
};
```

Another special use case for the return value of the setup is in jsx development, if the setup returns a function it will act as an h rendering function for rendering templates because JSX cannot write template tags like in a template.

Below is a calculator component example written in JSX:

```
import { ref, defineComponent } from 'vue';

export default defineComponent({
  setup() {
    const count = ref(0);

    const increment = () => {
      count.value++;
    };

    const decrement = () => {
      count.value--;
    };

    return (
      <div>
        <h1>Counter</h1>
        <p>Current count: {count.value}</p>
        <button onClick={increment}>Increase</button>
        <button onClick={decrement}>Decrease</button>
      </div>
    );
  }
});
```

The downside of this approach is that the Vue Devtools developer tool will not detect data defined in the setup function for display, such as count cannot see its value. The more common approach is to write the template in the render function, separating the template from the logic.

```
import { ref, defineComponent } from 'vue';

export default defineComponent({
  setup() {
    const count = ref(0);

    const increment = () => {
      count.value++;
    };

    const decrement = () => {
      count.value--;
    };

    return {
      count,
      increment,
      decrement
    };
  },

  render() {
    return (
      <div>
        <h1>Counter</h1>
        <p>Current count: {this.count}</p>
        <button onClick={this.increment}>Increase</button>
        <button onClick={this.decrement}>Decrease</button>
      </div>
    );
  }
});
```

### Reactive Functions

In Vue 3, reactive data refers to the data used in applications where the relevant views will automatically update to reflect changes when the data changes. The reactive implementation in Vue3 uses a Proxy to intercept and track data changes, so developers only need to manage and maintain application state and data, and can update page views without operating the underlying DOM.

Vue 3 provides some reactivity functions, such as `ref`, `reactive`, and `computed`, for defining reactive data.

- `reactive`: used to turn a normal JavaScript object into a reactive object. All properties become reactive

- `ref`: used to create a wrapper, making ordinary JavaScript data reactive, generally used to create basic data, and the underlying reactive is called to create object data. The variable created with the ref function needs to access its internal value through .value. template does not need to access through .value, because it is automatically destructed

- `computed`: used to create a computed property, which automatically updates based on the reactive data it depends on.

**Limitations of Using Reactive**

- reactive is only valid for object types (object, array, and Map, Set collection types), and is invalid for primitive types such as string, number, and boolean

- A reactive object cannot be arbitrarily “replaced”, as this will cause the responsive connection to the initial reference to be lost, if there is a replacement scenario, consider defining using ref, or as a reactive object property

```
let state = reactive({ count: 0 })
// The reference above ({ count: 0 }) will no longer be tracked (reactivity connection has been lost!)
state = { count: 1 }
// Replace with ref, reactivity is normal
let state1 = ref({ count: 0 })

state1.value = { count: 1 }
```

- reactive, props cannot be directly destructed, will lose the reactivity, the reason is that vue reactive tracking depends on the proxy object’s reference, destructing is equivalent to creating a new memory address, the reference changes. The solution is to use toRef, toRefs for value taking, or to assign the value to a reactive property.

A sample of reactive functions code

```
<template>
  <div>
    <h1>{{ fullName }}</h1>
    <input v-model="firstName" placeholder="firstName" />
    <input v-model="lastName" placeholder="lastName" />
  </div>
</template>

<script>
import { ref, computed, defineComponent } from 'vue';

export default defineComponent({
  setup() {
    const firstName = ref('');
    const lastName = ref('');

    const fullName = computed(() => {
      return `${firstName.value} ${lastName.value}`;
    });

    return {
      firstName,
      lastName,
      fullName
    };
  }
});
</script>
```

Analysis:

1. Using the ref function, we wrap firstName and lastName as reactive reference types, allowing their changes to be automatically tracked and updated in the view.

2. We use the computed function to create a full name computed attribute, fullName, which automatically calculates the full name based on the values of firstName and lastName. The computed function internally depends on firstName.value and lastName.value, and when their values change, fullName is automatically updated.

3. In the setup function, we expose firstName, lastName, and fullName as return values to the component instance so they can be accessed and used in the template.

### Computed Computation Functions

In Vue, we can use the computed function to create computed properties. A computed property is a special kind of reactive reference whose value is calculated by a function, and this function’s result is cached. When the dependencies of this function change, Vue automatically recalculates the value of this property and updates the view.

The computed property use scenario is generally used to optimize the computational logic of the template, such as class and style calculations depending on other data, reducing the writing of if-else templates, the calculation result is cached, reducing calculation and optimizing performance.

```
<template>
  <div>
    <h1 :class="headingClasses">Hello, Vue 3!</h1>
    <div :style="boxStyles"></div>
    <button @click="toggleColor">Toggle Color</button>
  </div>
</template>

<script>
import { ref, computed, defineComponent } from 'vue';

export default defineComponent({
  setup() {
    const isRed = ref(true);

    // Computed property: Return different class names based on the value of isRed
    const headingClasses = computed(() => {
      return {
        'red-text': isRed.value,
        'blue-text': !isRed.value
      };
    });

    // Computed property: Return different style objects based on the value of isRed
    const boxStyles = computed(() => {
      return {
        backgroundColor: isRed.value ? 'red' : 'blue',
        width: '200px',
        height: '200px'
      };
    });

    const toggleColor = () => {
      isRed.value = !isRed.value;
    };

    return {
      headingClasses,
      boxStyles,
      toggleColor
    };
  }
});
</script>
```

In the above example, we use the computed function to create two computed properties: headingClasses and boxStyles. Depending on the value of isRed, these two computed properties dynamically return different class names and style objects.

Points to note when using computed

- The getter should not have side effects: for example, do not make asynchronous requests or change the DOM in the getter, the getter’s responsibility should only be to calculate and return the value.

- Avoid directly modifying the computed property value, can define a setter

### Summarize the advantages of computed

- **Reactive**: The computed function will automatically track the reactive data it depends on, and when the dependent data changes, the computed function will recalculate and update the result. This ensures that the computed property’s value is always up-to-date.

- **Cached**: The computed function caches the calculated results, and only recalculates when the dependent data changes. This can avoid unnecessary calculations and improve performance.

- **Simplicity**: By using the computed function, complex logic can be encapsulated in the computed property, making the template more concise and readable. The result of the computed property can be directly used in the template, without having to write complex logic in the template.

- **Reusable**: Computed properties can be used multiple times within a component, improving the reusability of the code. If multiple components need the same calculation logic, the computed property can be defined in a function and referenced in multiple components.

**Notes on using computed**

- **Extra memory overhead**: The computed function will create a new reactive object to store the calculation result, which will occupy some memory space. If the computation property’s logic is complex or the computation result is large, it may lead to a large memory overhead.
- **Not suitable for asynchronous operations**: The computed function is only suitable for synchronous computation logic and is not suitable for handling asynchronous operations. If you need to perform asynchronous computation, you should use the watch function or async/await to handle it.

### Watch Listening Function

In Vue, we can use the watch function to create listeners. Listeners are special functions that monitor changes in some reactive references or computed properties and then perform some side effects. When the value of the reference or property being monitored changes, Vue will automatically execute the listener.

The watch function can monitor various data types such as objects, arrays, functions, etc.

```
// Listen to reference objects
watch(state.data, (newValue,oldValue) => {
  // Perform operation
})
// Listen to a specific value of an object, use a function
watch(() => state.data.id, (newValue,oldValue) => {
  // Perform operation
})
// Listen to multiple values, use an array
watch([data1, data2], ([newVal1,newVal2], [oldVal1,oldVal2]) => {
  // Perform operation
})
// Stop, call the callback function
const stop = watch(data, (newValue,oldValue) => {
  // Perform operation
})
// Stop listening
stop()
```

**Use case analysis of watch:**

1. Monitor changes in the form input

```
watch('formData', (newValue,oldValue) => {
  // Perform form validation operation
})
```

In the form, you can use the watch function to monitor changes in form data, and then perform form validation operations. When formData changes, the watch function will automatically execute the callback function.

2. Monitor changes in routing parameters

```
watch(route.params.id, (newValue,oldValue) => {
  // Perform page data update operation
})
```

When using Vue Router to route jumps, you can use the watch function to monitor changes in routing parameters and then perform page data update operations. When the routing parameters change, the watch function will automatically execute the callback function.

3. Monitor changes in asynchronous request results

```
watch('asyncData', (newValue,oldValue) => {
  // Perform page rendering operation
})
```

When making an asynchronous request, you can use the watch function to monitor changes in asynchronous request results and then perform page rendering operations. When asyncData changes, the watch function will automatically execute the callback function.

4. Monitor global status changes

```
watch(() => store.state.globalData,(newValue,oldValue) => {
  // Perform global status update operation
})
```

When using Pinia or Vuex for global state management, you can use the watch function to monitor changes in global status and then perform global status update operations. When globalData changes, the watch function will execute the callback function.

Other scenarios, such as

- Monitor changes in computed properties and perform corresponding operations

- Monitor changes in arrays and perform corresponding operations, changes in elements within the array, need to set the deep option to true (note, using deep consumes performance, use with caution)

**Advantages of watch**

- **More flexible**: The watch function of Vue 3 is more flexible compared to the method of Vue 2. It can monitor multiple data sources and can perform corresponding operations as needed, which makes it more convenient to handle complex data changes

- **Better performance**: Vue3’s watch function uses a responsive system based on Proxy, which has a better performance compared to Vue2’s Object.defineProperty. Proxy can directly intercept object reading, assignment, deletion, and other operations, thereby achieving finer-grained data change tracking, reducing unnecessary update operations, and improving performance

- **Better type inference**: Vue3’s watch function supports TypeScript, and can better perform type inference. Through type declaration, potential errors can be captured during the coding phase, improving the maintainability and readability of the code

**Use of watch to note**

- **Deeply nested data needs to be handled manually**: Vue 3’s watch function by default only listens for changes in the first layer properties of an object. If you need to listen to changes in deeply nested data, you need to manually set the deep option. This adds some extra code and handling logic

- **May cause repeat execution**: If you modify the data being monitored in the callback function, it may cause the callback function to be executed repeatedly. Avoid this situation when writing code, otherwise it may cause infinite loops or other unexpected results.

### Lifecycle Hook Function

Lifecycle hooks are special functions used to execute logic at different stages of a component’s lifecycle. They provide a mechanism for executing code when the component is created, mounted, updated, and unmounted.

Common lifecycle hook functions and their uses in Vue 3:

- **onBeforeCreate**: Called before the component instance is created. At this time, the component’s data, computed properties, and methods have not yet been initialized, and these properties and methods cannot be accessed.

- **onCreated**: Called after the component instance is created. At this point, the component’s data, computed properties, and methods have been initialized, but the DOM has not yet been rendered.

- **onBeforeMount**: Called before the component is mounted. At this time, the component’s template has been compiled, but it has not yet been rendered to the DOM.

- **onMounted**: Called after the component is mounted. At this time, the component has been rendered to the DOM, and operations such as DOM operations and asynchronous requests can be performed.

- **onBeforeUpdate**: Called before the component is updated. When the component’s data changes but has not been re-rendered, this hook function is triggered.

- **onUpdated**: Called after the component is updated. When the component’s data changes and re-rendering is complete, this hook function is triggered.

- **onBeforeUnmount**: Called before the component is unmounted. When the component is about to be destroyed, this hook function will be triggered. Operations such as clearing timers and canceling subscriptions can be performed here.

- **onUnmounted**: Called after the component is unmounted. Once the component has been destroyed and DOM nodes have been removed from the page, this hook function is triggered.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*GuPF9Iep5TpjcfNEfxcouA.png)

In the Composition API, combined with specific business scenarios, for example, in the onMounted hook function, you can perform DOM operations, subscribe to events, or send asynchronous requests. In the onBeforeUnmount hook function, you can clear resources, unsubscribe, or clear timer operations code.

```
<template>
  <div>
    <h1>{{ message }}</h1>
  </div>
</template>

<script>
import { ref, onMounted, onBeforeUnmount, defineComponent } from 'vue';

export default defineComponent({
  setup() {
    const message = ref('Hello, Vue 3!');

    onMounted(() => {
      // Operations executed after the component is mounted
      console.log('Component has been mounted');
      performDOMOperation();
      subscribeToEvent();
      fetchData();
    });

    onBeforeUnmount(() => {
      // Operations executed before the component is unmounted
      console.log('Component will unmount');
      cleanUpResources();
      unsubscribeFromEvent();
      clearTimer();
    });

    const performDOMOperation = () => {
      // Execute DOM operation
      const element = document.getElementById('my-element');
      // ...
    };

    const subscribeToEvent = () => {
      // Subscribe to event
      window.addEventListener('resize', handleResize);
    };

    const fetchData = async () => {
      // Send asynchronous request
      const response = await fetch('https://api.example.com/data');
      const data = await response.json();
      // ...
    };

    const cleanUpResources = () => {
      // Cleanup resources
      // ...
    };

    const unsubscribeFromEvent = () => {
      // Cancel event subscription
      window.removeEventListener('resize', handleResize);
    };

    const clearTimer = () => {
      // Clear the timer
      clearInterval(timer);
    };

    const handleResize = () => {
      // Handle the event callback
      // ...
    };

    let timer;
    onMounted(() => {
      // Operations executed after the component is mounted
      console.log('Component has been mounted');
      timer = setInterval(() => {
        // data update on a timer 
        message.value = 'Updated message';
      }, 1000);
    });

    onBeforeUnmount(() => {
      // Operations executed before the component is unmounted
      console.log('Component will unmount');
      clearInterval(timer);
    });

    return {
      message
    };
  }
});
</script>
```

In the `onMounted` hook function, the following operations are performed:

- **performDOMOperation**: Perform DOM operations, such as obtaining elements and operating styles.

- **subscribeToEvent**: Subscribe to events, such as window size adjustment events.

- **fetchData**: Send asynchronous requests, for example, obtaining remote data.

In the **onBeforeUnmount** hook function, we performed the following operations:

- **cleanUpResources**: Clean up resources, such as releasing memory, closing connections, etc.

- **unsubscribeFromEvent**: Cancel event subscriptions, such as canceling subscriptions to window size adjustment events.

- **clearTimer**: Clear the timer, for example, stop the timed data update operation.

By performing these operations in the appropriate lifecycle hook functions, we can ensure that necessary operations are performed at specific stages of the component’s life cycle and clean up before the component is destroyed. This can improve the reliability and performance of the code and avoid possible memory leaks and unnecessary resource occupancy.

## Best Practices for the Composition API

To write high-quality code, when using Vue 3’s Composition API, I’ve summarized some good practices based on my own development experience.

### Composition API Coding Standards

1. Definition of reactive data

Reactive data defined in the setup function should be placed at the top of the function, and not defined within loops or conditional statements. This ensures the accuracy and consistency of the reactive data.

```
import { reactive, ref } from 'vue';
setup() {
  const state = reactive({
    name: 'John',
    age: 30
  });
  return {
    state
  };
}
```

2、Writing order

In order to reduce too flexible writing in the setup function, leading to messy code logic. A standard writing order is recommended, proposed as follows: reactive functions, computed functions, watch listener functions, lifecycle hooks, custom methods, etc. This makes the code more readable and maintainable within the team.

```
import { ref, computed, watch, onMounted, onBeforeUnmount, defineComponent } from 'vue';
export default defineComponent({
  setup() {
    // Reactive data
    const firstName = ref('John');
    const lastName = ref('Doe');
    const age = ref(30);
    // Computed properties
    const fullName = computed(() => `${firstName.value} ${lastName.value}`);
    // Observer
    watch(
      age,
      (newAge, oldAge) => {
        console.log(`Age has changed from ${oldAge} to ${newAge}`);
      }
    );
    // Lifecycle hooks
    onMounted(() => {
      console.log('Component has been mounted');
      // You can perform some initialization operations here
    });
    onBeforeUnmount(() => {
      console.log('Component will unmount');
      // Resources can be cleaned or subscriptions cancelled here
    });
    // Method
    const increaseAge = () => {
      age.value++;
    };
    // Return data and methods
    return {
      firstName,
      lastName,
      age,
      fullName,
      increaseAge
    };
  }
});
```

### Custom Hooks

Custom Hooks draw on React’s ideas. Vue 3’s Composition API’s flexibility and composition can help us encapsulate reusable logic and share it across different components.

For instance, in a business scenario where multiple components need to obtain user geolocation information, we can create a custom Hook called useGeolocation to encapsulate the logic of obtaining geolocation, so it can be reused in multiple components when needed.

Here’s a code example:

```
<template>
  <div>
    <p>Latitude: {{ latitude }}</p>
    <p>Longitude: {{ longitude }}</p>
  </div>
</template>
<script>
import { ref, onMounted, onBeforeUnmount, defineComponent } from 'vue';
// Custom Hook
function useGeolocation() {
  const latitude = ref(null);
  const longitude = ref(null);
  const successCallback = (position) => {
    latitude.value = position.coords.latitude;
    longitude.value = position.coords.longitude;
  };
  const errorCallback = (error) => {
    console.error('Failed to get geolocation:', error);
  };
  onMounted(() => {
    // Get geolocation
    navigator.geolocation.getCurrentPosition(successCallback, errorCallback);
  });
  onBeforeUnmount(() => {
    // Cancel geolocation acquisition
    navigator.geolocation.clearWatch(watchId);
  });
  return {
    latitude,
    longitude
  };
}
export default defineComponent({
  setup() {
    const { latitude, longitude } = useGeolocation();
    return {
      latitude,
      longitude
    };
  }
});
</script>
```

In the useGeolocation function, we create the latitude and longitude reactive data with ref. In the successCallback callback, we get the user’s geolocation and assign the latitude and longitude values to the corresponding reactive data.

In the onMounted hook, we call the navigator.geolocation.getCurrentPosition method to get the geolocation. In the onBeforeUnmount hook, we call the navigator.geolocation.clearWatch method to cancel the geolocation acquisition.

By doing this, we can reuse the logic of obtaining geolocation in multiple components without having to rewrite the same code in each component. This enhances the reusability and maintainability of the code and makes the logic clearer.

## Summary of the Composition API

Finally, let’s summarize the best practices for developing with the Composition API:

1. **Single responsibility principle**: Group related logic and state into a single custom function to make the code clearer and more maintainable.

2. **Use reactive functions**: Use ref to wrap basic data types and reactive to wrap objects or arrays for reactive tracking.

3. **Use computed**: Create computed properties using the computed function to dynamically calculate values based on dependent data, avoiding redundant calculations and manual dependency tracking.

4. **Use watch**: Use the watch function to observe changes in reactive data and execute corresponding operations, such as sending network requests, updating states, etc.

5. **Combine multiple functions**: Logic can be composed by calling multiple custom functions, making the code more reusable and testable.

6. **Use provide and inject**: Use the provide function in the parent component to provide data, and then use the inject function in the child component to inject these data to achieve cross-component data sharing.

7. **Utilize lifecycle hooks**: Use lifecycle hooks such as onMounted, onBeforeUnmount, etc., to execute corresponding operations, such as subscribing to events, sending requests, etc.

8. **Separate side-effect code**: Place code that has side effects (such as timers, network requests, etc.) in onMounted and onBeforeUnmount hooks to ensure correct initialization and cleanup.
