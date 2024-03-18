---
title: 'Use and Encapsulation Concept of Custom Hooks Functions'
date: 2024-02-04T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*B-rRsR1EO03Ui8Dn7a9XsQ.png"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: "Hooks can be understood as common methods joining vue3 api."
---

## What are hooks?

In Vue3, hooks are a kind of function writing style, which essentially encapsulates certain independent functional Javascript code from a file.

The main purpose of it is that Vue3 draws from a mechanism in React, which is used to share state logic and side effects in function-based components, thereby improving code reusability.

Note: Hooks are somewhat similar to mixins in Vue2, but compared to mixins, hooks make it clearer where the reused code originates from, making it easier to understand.

## Advantages of hooks

Hooks encapsulate components as independent logic. Its internal attributes, functions, etc. have a reactive effect that’s associated to external components.

The function of custom hooks is similar to the mixin technology used in Vue2, which is convenient and easy to use.

It is highly cohesive and loosely coupled, and it is reusable when encapsulated with Vue3’s combination API.

## Standards that custom hooks must meet

They should have reusable functionality, only then they need to be extracted as independent hooks files.

The function name or file name should start with ‘use’, such as: useXX.

When importing, reactive variables or methods should be explicitly destructured and exposed.

Examples are as follows:

```
const { nameRef,Fn }= useXX()
```

## Difference between hooks and utils

Similarities: Through encapsulation with hooks and utils functions, code sharing and reuse can be achieved between components, which improves the reusability and maintainability of code.

### Differences:

Different presentations: Hooks wrap things at the component level (hook functions, etc.) based on utils; utils are generally used to encapsulate logic functions, without component-related stuff.

Reactions to data: If ref, reactive, computed, these APIs in hooks are involved, the data is reactive; while utils are just plain extraction of common methods and thus are not reactive.

Different scopes of application: Hooks encapsulation can extract component status and lifecycle methods, which can be shared and reused across multiple components; utils generally refer to auxiliary functions or tool methods used to implement common operations or provide specific functionality.

### Summary:

Utils are general-purpose utility functions, while hooks are a kind of encapsulation of utils, used to share state logic and side effects in components.

By using hooks, you can simplify your code and make it more readable and maintainable.

## Difference between hooks and mixin

**Similarities**: Hooks and Mixins are both common means to extract code logic, convenient for code reuse;

**Differences**:

- Different syntax and usage: Hooks are a way of functional programming introduced in the Composition API in Vue3, while Mixins are an object mixing mechanism in Vue2. Hooks are defined and used as functions, while Mixins are defined and applied by objects.

- Different combinations and flexibility: Hooks allow developers to combine code according to logic, encapsulate it as a custom Hook function, and improve the reuse rate of code. Meanwhile, the properties and methods of Mixins in components will be merged with the properties and methods of components, and may cause naming conflicts or unpredictable behaviors.

- Different reactive systems: Upgrading to Vue 3 comes with a new reactive system from Composition API that creates reactive data through reactive and ref and can more accurately control component updates and dependency tracking. Mixins use Vue 2’s reactive system, which keeps tracking and updating data more simple, but there may be some performance problems.

- Different Lifecycle Hooks: Instead of lifecycle hooks in Vue 2, you can use hook functions like onMounted, onUpdated, etc. in Vue 3’s Composition API to manage the lifecycle of components more flexibly.

### Advantages and disadvantages of mixins

**Advantages**: Code logic reuse in components;

**Disadvantages**:

- Source of variables is unclear: The source of variables is not clear (implicitly passed in), which is not conducive to reading and makes the code hard to maintain.

- Naming conflicts: The lifecycles of multiple mixins will fuse and run together, but properties and methods with the same name cannot be fused and may cause conflicts.

- Overuse leads to maintenance problems: Many-to-many relationships can exist between mixins and components, and the complexity is higher (i.e., a component can reference multiple mixins, and a mixin can be referenced by multiple components).

**Note**: The Composition API proposed by VUE3 is aimed at solving these problems. The disadvantages of mixins are one of the main motivations behind the Composition API, which is inspired by React Hooks.

### Hooks code:

Example of useCount.ts function:

```
import { ref, onMounted, computed } from 'vue';
 
export default function useCount {
  
    const count = ref(0);   

    const doubleCount = computed(
      () => count.value * 2
    );
    
     const increase = (delta) => {
        return count.value + delta;
    }    
    
    return {
      count,
      doubleCount,
      increase  
    };
 
}
```

useCount invoked in component:

```
<script setup lang="ts">
    import useCount from "@/hooks/useCount";
    const { count,doubleCount,increase } = useCount;
    const newCount = increase(10); // output: 10       
</script>
```

### Mixins code:

```
export default const countMixin = {
  data() {
    return {
      count: 0
    };
  },
  computed: {
    doubleCount() {
      return this.count * 2;
    }
  },
  methods: {
      increase(delta){
        return this.count + delta;
    }    
};
```

Mixins invoked in component:

```
import countMixin from '@/mixin/countMixin';
export default {
  mixins: [countMixin],
  mounted() {
    console.log(this.doubleCount);// output: 0
    const newCount = this.setIncrease(10); // output: 10
  },
  methods: {
      setIncrease(count){
        this.increase(count)              
      }
  }
}
```

These two examples display the difference in the code style and organization between using Hooks and Mixins. Hooks apply functional programming logic and states, while Mixins combine and share codes in an object-oriented way.

Vue3 custom Hooks apply at the function scope under components, while in Vue2 era, Mixins apply at a global scope under components. The global scope can sometimes be uncontrolled. Just like the var and let variable declaration keywords, const and let are corrections of var. Composition API indeed corrects the high coupling issue and the ubiquitous black box of ‘this’ in Vue2 era Option API. Vue3 custom Hooks are a step forward.

## Hooks function encapsulation example

### Example 1: Data Exporting (useDownload)

useDownload function encapsulation:

```
import { ElNotification } from "element-plus";
/**
@description Receive data stream to generate a blob, create a link, download file
@param {any} data Exported file blob data (Required)
@param {String} tempName Exported file name (Required)
@param {Boolean} isNotify Whether export message is prompted (Default is true)
@param {String} fileType Exported file format (Default is .xlsx)
*/
interface useDownloadParam {
  data: any;
  tempName: string;
  isNotify?: boolean;
  fileType?: string;
}

export const useDownload = async ({ data, tempName, isNotify = true, fileType = ".xlsx" }: useDownloadParam) => {
  if (isNotify) {
    ElNotification({
      title: "Kind reminder",
      message: "Large data size might slow down the downloading process, please be patient!",
      type: "info",
      duration: 3000
    });
  }
  try {
    const blob = new Blob([data]);
    // Combat edge not supporting createObjectURL method
    if ("msSaveOrOpenBlob" in navigator) return window.navigator.msSaveOrOpenBlob(blob, tempName + fileType);
    const blobUrl = window.URL.createObjectURL(blob);
    const exportFile = document.createElement("a");
    exportFile.style.display = "none";
    exportFile.download = ${tempName}${fileType};
    exportFile.href = blobUrl;
    document.body.appendChild(exportFile);
    exportFile.click();
    // Eliminate url's impact on download
    document.body.removeChild(exportFile);
    window.URL.revokeObjectURL(blobUrl);
  } catch (error) {
    console.log(error);
  }
};
```

useDownload used in component:

```
<script setup lang="ts">
import { useDownload } from "@/hooks/useDownload";
const userForm = reactive({})
const userListExport = () => {
  new Promise(resolve => {
    $Request({
      url: $Urls.userListExport,
      method: "post",
      data: userForm,
      responseType: "blob"
    }).then((res: any) => {
      useDownload({
        data: res.data,
        tempName："User List"
      });
      resolve(res);
    });
  });
};
</script>
```

### Example 2: Addition and Subtraction Count (useCount)

useCount function encapsulation:

```
import { computed, ref, Ref } from "vue"
// Define hook methods
type CountResultProps = {
    count:Ref<number>;
    multiple:Ref<number>; // Computed properties
    increase:(delta?:number)=>void;
    decrease:(delta?:number)=> void;
}

export default function useCount(initValue = 1):CountResultProps{
    const count = ref(initValue)
    
    const multiple  = computed(
      ()=>count.value * 2
    )
    
    const increase = (delta?:number):void =>{
        if(typeof delta !== 'undefined'){
            count.value += delta
        }else{
            count.value += 1
        }
    }
   
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
```

useCount function used in the component:

```
<template>
   <p>count:{{count}}</p>
   <p>times:{{multiple}}</p>
   <div>
     <button @click="increase(1)">Increase by one</button>
     <button @click="decrease(1)">Decrease by one</button>
     // Use the method in hooks directly as a callback function in the template
   </div>
</template>

<script setup lang="ts">
    import useCount from "@/hooks/useCount"
    const {count,multiple,increase,decrease}  = useCount(10)
</script>
```

### Example 3: Obtain Mouse Trigger Point Coordinate (useMousePosition)

useMousePosition function encapsulation:

```
import { ref, onMounted, onUnmounted, Ref } from 'vue'

interface MousePosition {
  x: Ref<number>,
  y: Ref<number>
}

export default function useMousePosition(): MousePosition {
  const x = ref(0)
  const y = ref(0)

  const updateMouse = (e: MouseEvent) => {
    x.value = e.pageX
    y.value = e.pageY
  }

  onMounted(() => {
    document.addEventListener('click', updateMouse)
  })

  onUnmounted(() => {
    document.removeEventListener('click', updateMouse)
  })

  return { x, y }
}
```

useMousePosition used in component:

```
<template>
  <div>
    <p>X: {{ x }}</p>
    <p>Y: {{ y }}</p>
  </div>
</template>
<script lang="ts">
    import useMousePosition from '@/hooks/useMousePosition'
    const { x, y } = useMousePosition();
</script>
```

### Hooks function encapsulation details are summarized

1. How Hook functions receive parameters:

- Hooks function receives parameters via props, first define parameter type, then destructure internally.

```
export function commonRequest(params: Axios.AxiosParams) {
    let {
        url,
        method,
        data,
        responseType = "json",
    } = params;
  }
```

- Receives parameters object, first set default values, then define parameter type.

```
interface  DeprecationParam {
    from:string;
    replacement:string;
    type:string;
}
export const useDeprecated = (
  { from, replacement,type = 'API' }: DeprecationParam,
) => {}
```

2. Destructuring rename method.

```
// setup

const { list: goodsList, getList: getGoodsList } = useList(
  axios.get('/url/get/goods')
)
const { list: recommendList, getList: getRecommendList } = useList(
  axios.get('/url/get/recommendGoods')
)
```

3. KeyboardEvent is the type for mouse key event.

```
export const useEscapeKeydown = (handler: (e: KeyboardEvent) => void) => {}
```

## Summary

In the Vue2 era with Option API, data, methods, watch etc. are written separately, which makes the code fragmented and scattered. Once the code becomes complicated, it can easily lead to high coupling, making the code switching back and forth during maintenance cumbersome!

In the Vue3 era with Composition API, through the use of various Hooks and custom Hooks, fragmented reactive variables and methods are written in blocks according to functionality, achieving high cohesion and low coupling.
