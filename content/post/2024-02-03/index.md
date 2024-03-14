---
title: 'Understanding Hooks in Vue3 (Why use hooks)'
date: 2024-02-03T16:22:38+08:00
draft: true
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*B-rRsR1EO03Ui8Dn7a9XsQ.png"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: "Hooks can be understood as common methods joining vue3 api."
---

## Prerequisite Knowledge

### What is a mixin?

Mixin, translated, means mixing in. It’s not just in the Vue framework that mixin exists, to be precise, mixin is a concept, the concept of mixing in. The content of the mix can be used where it is mixed in, and it will automatically distribute the mixed elements accurately to the specified components. In Vue, mixin is equivalent to putting the specified variables & functions where they would be if they were not mixed in. It can be considered that mixin in Vue is equivalent to a component within a component.

For example: Currently, the logic that needs to be handled in the watch in component A is hanldleParams, and the watch in component B also needs such logic — hanldleParams. So how should we abstract these two same logics for reuse?

There are two methods: (The difference between these two methods represents the difference between mixin and utils)

1. Extract Function: The first method is to extract hanldleParams as a function and call hanldleParams in the watch;

2. Mixin: Although the previous method of extracting functions can solve some reuse problems, we still need to write watch in the component, after all, both components are calling in the watch hook. If every component writes watch, then watch is also something repeated, so mixin is the component that can extract all the watch hooks, that is to say, mixin can extract not only pure function logic, but also can extract the unique hooks of Vue components and other logic, achieving further reuse, this is the role of mixin. So components A\B share a watch through mixin, just import it, and developers don’t need to place it in a specific location.

Features: The data and methods in Mixin are independent, and they do not affect each other after being used in components.

### What problem does mixin solve?

Mixin resolves two types of reuse:

1. Reuse of logical functions

2. Reuse of Vue component configuration

Note: Component configuration reuse refers to the options API (e.g. data, computed, watch) or life cycle hooks (created, mounted, destroyed) in the component.

### Use & Use Scene?

**Key**: In vue, mixin is defined as an object, with the corresponding options API and life cycle hooks of the vue component placed in the object.

```
export const mixins = {
  data() {
    return {};
  },
  computed: {},
  created() {},
  mounted() {},
  methods: {},
};
```

**Remember**: Mixins generally exist in the options API and component lifecycle hooks of Vue components because the abstraction of functions also needs to be placed in specific APIs or hooks in the component. Therefore, mixin considers this and directly configures all APIs as long as they are placed in the function.

That is to say, except for the template in the component, any other options-API can be extracted and placed in mixin (all the logic in Vue2 is nothing more than data, methods, computed, watch, these can be placed in mixin as they are)

For example, to abstract a hanldleParams function, we usually export it in the config file and then import it into the component for use, use in the methods in data to process the data, and we need to set the variable b in data, then this logic can be extracted and placed in mixin. As for the data or logic related to component business, they are generally written in the component.

For example:

```
// Define a mixin
export const mixins = {
  data() {
    return {
      msg: "xxx",
    };
  },
  computed: {},
  created() {
    console.log("the created lifecycle function in mixin");
  },
  mounted() {
    console.log("the mounted lifecycle function in mixin");
  },
  methods: {
    clickMe() {
      console.log("the click event in mixin");
    },
  },
};
//

// The exported mixins are used in src/App.vue
<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" />
    <button @click="clickMe">click</button>
  </div>
</template>

<script>
import { mixins } from "./mixin/index";
export default {
  name: "App",
  mixins: [mixins], // Register mixin, so that all the hook functions in mixin are equivalent to the hooks in the component
  components: {},
  created(){
    console.log("Component calls minxi data",this.msg);
  },
  mounted(){
    console.log("the mounted lifecycle function of the component")
  }
};
</script>
```

The above code is equivalent to:

```
// Use the exported mixin in src/App.vue
<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" />
    <button @click="clickMe">clickMe</button>
  </div>
</template>

<script>
import { mixins } from "./mixin/index";
export default {
  name: "App",
  mixins: [mixins], // Register mixin, so all the hook functions in mixin are equivalent to the hooks in the component
  data() {
    return {
      msg: "xxx",
    };
  },
  // The original component does not have a methdos method, the method in mixin is directly placed in the component
  // Note: In Vue, when the method name in methods conflicts with the method in mixin, the solution is that the priority of the component is higher than that of mixin
  methods: {
    clickMe() {
      console.log("the click event in mixin");
    },
  },
  created(){
    console.log("the created lifecycle function in mixin");
    // The created hook in mixin is more prioritized than the component in the lifecycle
    console.log("Component calls minxi data",this.msg);
  },
  mounted(){
    // In mixin's mounted hook, the lifecycle of mixin's hook has higher priority than the component's priority
    console.log("the mounted lifecycle function in mixin");
    console.log("the mounted lifecycle function of the component")
  }
};
</script>
```

**Note**: The priority of the same hooks in mixin and vue components:

- The lifecycle functions in mixin will be merged and executed with the lifecycle functions of the components.

- The data in mixin can also be used in components.

- The methods in mixin can be directly called inside the component.

- After the lifecycle functions are merged, the execution order is: execute the one in mixin first, and then the one in the component.

In addition, the import of mixin for different components will not affect each other’s data. For example:

```
// mixin
export const mixins = {
  data() {
    return {
      msg: "xxx",
    };
  },
  computed: {},
  created() {
    console.log("the created lifecycle function in mixin");
  },
  mounted() {
    console.log("the mounted lifecycle function in mixin");
  },
  methods: {
    clickMe() {
      console.log("the click event in mixin");
    },
  },
};
```

Use mixin in the app file and use the changeMsg function to change the variable msg in the data of mixin. At this time, the variable msg used by other components remains unchanged.

```
<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/logo.png" />
    <button @click="clickMe">clickMe</button>
    <button @click="changeMsg">Change mixin data</button>
    <demo></demo>
  </div>
</template>

<script>
import { mixins } from "./mixin/index";
import demo from "./components/demo.vue";
export default {
  name: "App",
  mixins: [mixins],
  components: { demo },
  created() {
    console.log("Component calls minxi data", this.msg);
  },
  mounted() {
    console.log("the mounted lifecycle function in the component");
  },
  methods: {
    changeMsg() {
      this.msg = "xxxxx";
      console.log("Change the msg after:", this.msg);
    },
  },
};
</script>
```

### What are the existing disadvantages?

Advantages: Reuse of hook function registration in components

Disadvantages:

- There will be conflicts if the same function name is registered in the same hook (In Vue, the solution to conflicts is that the priority in the component is higher than that in mixin)

- Error localization takes time

- Abuse will result in maintenance issues

## What are hooks?

Generally speaking, in our development, we will automatically abstract logic functions into utils. Utils only contain pure logic, not related to components, such as pure functions defined in methods. Hooks are a layer of component-level things (such as hook functions) based on utils. For example: every time we click a button, a pop-up window will appear and automatically display the current date. If I put the function in the util, every time I reuse it, I need to put the date function in click=handleClick. Through the handleClick function to manage utils, I might as well wrap handleClick as well, and call it directly next time for reusing the methods registration process hooks and utils difference: if the data of ref,reactive,computed and other APIs is involved in hooks, the data is reactive, while utils simply extract common methods and do not have reactivity. Therefore, hooks can be understood as common methods joining vue3 api.

## Why are there hooks?

So hooks are equivalent to logic encapsulation at the component level. This logic encapsulation can also be implemented in vue2’s mixin. Why use hooks?

Difference between hooks and mixin:

**mixin embodies the options API, while the other embodies the composition API**

1. mixin

- There is something in vue2: Mixins can achieve this

- mixins are the extraction of these multiple identical logics, each component only needs to introduce mixins to achieve code reuse

- Fault one: There will be issues of overriding

- The data, methods, filters of the component will overwrite the same-name data, methods, and filters in mixins

- Fault two: Implicit import, the source of variables is not clear, not conducive to reading, making the code hard to maintain

- Fault three: Mixin cannot pass in flexible parameters, for example (Example 1 of Fault 3):

I need to define a variable name, but the initial value of the name is random. When the name is defined in mixin, its initialization must be fixed. If we want to change, we can only register a method in method to modify the value of the name:

Example 1: Example of using mixin

```
// Mixin file：name-mixin.js
export default {
  data() {
    return {
      name: 'zhng'
    }
  },
  methods: {
    setName(name) {
      this.name = name
    }
  }
}

// Component：my-component.vue
<template>
  <div>{{ name }}</div>
<template>
<script>
import nameMixin from './name-mixin';
export default {
  mixins: [nameMixin],
  mounted() {
    setTimeout(() => {
      this.setName('Tom') 
      // Setting the parameter value by calling setName in the component, unable to pass parameters, which limits the flexibility of Mixin
    }, 3000)
  }
}
<script>
```

Example 2: Example of using hooks

```
import { computed, ref, Ref } from "vue"
// Define the hook method
type CountResultProps = {
    count:Ref<number>;
    multiple:Ref<number>;
    increase:(delta?:number)=>void;
    decrease:(delta?:number)=> void;
}
export default function useCount(initValue = 1):CountResultProps{
    const count = ref(initValue)
    const increase = (delta?:number):void =>{
        if(typeof delta !== 'undefined'){
            count.value += delta
        }else{
            count.value += 1
        }
    }
    const multiple  = computed(()=>count.value * 2)
    const decrease = (delta?:number):void=>{
        if(typeof delta !== "undefined"){
            count.value -= delta
        }else{
            count.value -= 1
        }
    }
    return {
        count,
        increase,
        decrease,
        multiple
    }
 
}

// The usage in the component
<template>
   <p>count:{{count}}</p>
   <p>Multiplier:{{multiple}}</p>
   <div>
     <button @click="increase(10)">Increase One</button>
     <button @click="decrease(10)">Decrease One</button>
      // Use the methods in hooks directly as callback functions in the template
   </div>
</template>
<script setup lang="ts">
import useCount from "../views/Hook"
const {count,multiple,increase,decrease}  = useCount(10)
</script>
<style>
 
</style>
```

Example 3: Key points example for beginners using hooks

*Hooks generally export a function, such as: I need hooks to export a common name variable and setName function*

```
import {ref} from 'vue'
// Export a name_hooks.ts file
// No need to write in setup in hooks
export const name_hooks = function(value: string) {
   const name = ref('')
   const setName = (value: string) => {
       name.value = value
   }
   return {
      name, setName
   }
}
// import hooks file
<template>
    <div>{{ name }}</div>
    <select @change="setName"></select>
    // Here the change event of the select component will automatically pass the value
    // Then the value acts as a parameter passed to setName
</template>
import { name_hooks } from './name_hooks'
export default defineComponent({
    setup() {
       const { name, setName } = name_hooks()
      // Note: Usually need to destructure the desired properties and methods into the component
       return {
          name, setName
       }
    }
})
```

The above hooks usage method, common operations are:

The exported hooks is a function, in which ref, reactive can be used, so hooks defined variables and methods are the same as in the component

The hooks function generally returns an object, the object is a two-way binding variable, the first thing when being referenced in vue is to destructure (be aware of potential pitfalls in vue3)

2. hooks:

- In Vue3, we can: Custom Hook

- Vue3’s hook function is equivalent to vue2’s mixin, but: hooks are functions

- Vue3’s hook function can help us improve the reusability of the code, allowing us to use hooks function in different components.

## Usage scenario of hooks — Custom hook

1. Common business use scenarios in Hooks

Reinventing the wheel components, apart from some unnecessary repetitions, some functional components indeed need to be encapsulated. For example, some drop-down select boxes that need to request backend dictionary to display in the frontend, buttons that need to show loading status after being clicked, forms with query conditions, these very common business scenarios, we can encapsulate into components. However, when encapsulated into components, we will encounter the problems mentioned earlier. Each person’s usage habits and encapsulation habits are different, making it difficult to please everyone. In such scenarios, hooks can be the solution.

2. Standards that a custom hook needs to meet

Extract reusable functionality into an external js file.
Start the function name and filename with “use”
When referencing, explicitly destructure the reactive variables or methods, such as: const { nameRef, fun } = useXXX() (Destructure the variables and methods of the custom hooks in setup).

3. Examples of using custom hooks

Example 1

Custom hooks-1 : useCut

```
import {ref,watch} from "vue"
export function useCut({num1,num2}){
   const cutNum = ref(0);
   watch([num1,num2],(num1,num2)=>{
    cutFunc(num1,num2)
  })
  const cutFunc = (num1,num2)=>{
    cutNum.value = num1+num2
  }
  return {
    cutNum,
    cutFunc
  }
}
```

hooks2

```
import {ref,watch} from "vue"
const useAdd = ({num1,num2})=>{
  const addNum = ref(0);
  watch([num1,num2],(num1,num2)=>{
    addFunc(num1,num2)
  })
  const addFunc = (num1,num2)=>{
    addNum.value = num1+num2
  }
  return {
    addNum,
    addFunc
  }
}
export default useAdd
```

hooks3

```
<template>
    <div>
        num1:<input v-model.number="num1" style="width:100px" />
        <br />
        num2:<input v-model.number="num2" style="width:100px" />
    </div>
    <span>Addition equals:{{ addNum }}</span>
    <br />
    <span>Subtraction:{{ cutNum }}</span>
</template>
import { ref } from 'vue'
import useAdd from './addHook.js'     //Import auto hook 
import { useCut } from './cutHook.js' //Import auto hook 
const num1 = ref(2)
const num2 = ref(1)
const { addNum, addFunc } = useAdd({ num1, num2 })
// Addition function - Custom Hook (expose reactive variables or methods)
// Since hooks are functions, unlike mixins which are objects, they make it easier to pass data variables into components for abstract logic to use.
addFn(num1.value, num2.value)
const { cutNum, cutFunc } = useCut({ num1, num2 })
// Subtraction function - Custom Hook (expose reactive variables or methods)
subFn(num1.value, num2.value)
```
