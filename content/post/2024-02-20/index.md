---
title: 'Understanding watch and watchEffect in Vue3'
date: 2024-02-20T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*v6hSD9YqWXc8J6oR-4kQsQ.png"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

## Differences between watch and watchEffect in Vue3

### Features of watch:

Watch listening function can add configuration options, or it can be configured as empty. When the configuration option is empty, the characteristics of watch are as follows:

- **Lazy**: it does not execute immediately when it runs

- **More specific**: you need to add the property to listen to

- **Access to previous values the property**: the callback function will return the latest value and the value before modification

- **Configurable**: configuration items can supplement the shortcomings of watch functions

  (1) immediate: configure whether the watch property executes immediately. When the value is true, it executes immediately once it runs. When the value is false, it remains lazy

  (2) deep: configure whether watch listens deeply. When the value is true, it can listen to all properties of the object. When the value is false, it remains more specific and must be specified on the specific property

### Features of watchEffect

- **Non-lazy**: it executes immediately as soon as it runs

- **More abstract**: you don’t need to specify who to listen to when using it. You can directly use it in the callback function. Whoever you use, you listen to

- **Cannot access previous values**: can only access the current latest value, cannot access the value before modification

- **Manually stop listening**: there is a return value, the return value is the function to stop listening, you can stop listening directly when calling

- **Clear side effects**: click the button multiple times, and only execute once, a bit like function debounce

  (1) Clicking the pagination button has a network problem, the requested data and the number of pages do not match. This can be achieved by clearing the side effects

- **Listener debugging**: two options onTrack and onTrigger can be used to debug the behavior of listeners, but they can only be used in development mode

  (1) onTrack will be called when a responsive property or ref is tracked as a dependency

  (2) onTrigger will be called when a dependency change triggers a side effect

Both of these callbacks will receive a debugger event containing information about the dependency.

---

## Usage

### watch

**Syntax**

```
import { watch } from "vue"
watch(
  name, // the property that needs to be observed
  (curVal, preVal)=>{ ... }, // is an arrow function, which represents the latest value observed and the value before the current modification, where logical processing takes place.
  options // configuration options, configuration of the watcher, e.g., whether to deep watch.
);
```

**Monitor the reactive data defined by ref.**

```
<template>
  <div>
    <div>value：{{count}}</div>
    <button @click="add">change</button>
  </div>
</template>

<script>
import { ref, watch } from 'vue';
export default {
  setup(){
    const count = ref(0);
    const add = () => {
      count.value ++
    };
    watch(count,(newVal,oldVal) => {
      console.log('The value has changed:',newVal,oldVal)
      // The value has changed: 1 0
    })
    return {
      count,
      add,
    }
  }
}
</script>
```

**Monitor the reactive data defined by reactive.**

```
<template>
  <div>
    <div>{{obj.name}}</div>
    <div>{{obj.age}}</div>
    <button @click="changeName">change</button>
  </div>
</template>

<script>
import { reactive, watch } from 'vue';
export default {
  setup(){
    const obj = reactive({
      name:'zs',
      age:14
    });
    const changeName = () => {
      obj.name = 'ls';
    };
    watch(obj,(newVal,oldVal) => {
      console.log('The value has changed:',newVal,oldVal)
      // The value has changed: Proxy {name: 'ls', age: 14} Proxy {name: 'ls', age: 14}
    })
    return {
      obj,
      changeName,
    }
  }
}
</script>
```

**Listening to multiple reactive data.**

```
<template>
  <div>
    <div>{{obj.name}}</div>
    <div>{{obj.age}}</div>
    <div>{{count}}</div>
    <button @click="changeName">change</button>
  </div>
</template>

<script>
import { reactive, ref, watch } from 'vue';
export default {
  setup(){
    const count = ref(0);
    const obj = reactive({
      name:'zs',
      age:14
    });
    const changeName = () => {
      obj.name = 'ls';
    };
    watch([count,obj],() => {
      console.log('The multiple data being monitored have changed.')
    })
    return {
      obj,
      count,
      changeName,
    }
  }
}
</script>
```

**Monitoring changes in a property of an object.**

```
<template>
  <div>
    <div>{{obj.name}}</div>
    <div>{{obj.age}}</div>
    <button @click="changeName">change</button>
  </div>
</template>

<script>
import { reactive, watch } from 'vue';
export default {
  setup(){
    const obj = reactive({
      name:'zs',
      age:14
    });
    const changeName = () => {
      obj.name = 'ls';
    };
    watch(() => obj.name,() => {
      console.log('The obj.name being monitored has changed.')
    })
    return {
      obj,
      changeName,
    }
  }
}
</script>
```

**deep、immediate**

```
<template>
  <div>
    <div>{{obj.brand.name}}</div>
    <button @click="changeBrandName">change</button>
  </div>
</template>

<script>
import { reactive, ref, watch } from 'vue';
export default {
  setup(){
    const obj = reactive({
      name:'zs',
      age:14,
      brand:{
        id:1,
        name:'bwm'
      }
    });
    const changeBrandName = () => {
      obj.brand.name = 'benz';
    };
    watch(() => obj.brand,() => {
      console.log('The obj.brand.name being monitored has changed.')
    },{
      deep:true,
      immediate:true,
    })
    return {
      obj,
      changeBrandName,
    }
  }
}
</script>
```

### watchEffect

watchEffect is also a frame listener, is an effect function.

It listens to all properties of reference data types, without needing to specify a certain property. Once it runs, it will immediately start listening, and stop listening when the component is unloaded.

**Systax**

```
<template>
  <div>
    <input type="text" v-model="obj.name"> 
  </div>
</template>

<script>
import { reactive, watchEffect } from 'vue';
export default {
  setup(){
    let obj = reactive({
      name:'zs'
    });
    watchEffect(() => {
      console.log('name:',obj.name)
    })

    return {
      obj
    }
  }
}
</script>
```

**Stop listening**

When watchEffect is called in a component’s setup() function or lifecycle hook, the listener will be linked to the component’s lifecycle and automatically stop when the component is unloaded.
In some cases, the return value can also be explicitly called to stop listening:

```
<template>
  <div>
    <input type="text" v-model="obj.name"> 
    <button @click="stopWatchEffect">Stop listening</button>
  </div>
</template>

<script>
import { reactive, watchEffect } from 'vue';
export default {
  setup(){
    let obj = reactive({
      name:'zs'
    });
    const stop = watchEffect(() => {
      console.log('name:',obj.name)
    })
    const stopWatchEffect = () => {
      console.log('Stop listening')
      stop();
    }

    return {
      obj,
      stopWatchEffect,
    }
  }
}
</script>
```

**Clearing side effects**

Sometimes effect functions may perform some async side effects, these responses may need to be cleared when they become invalid. Scenario: There is a paging component with 5 pages, clicking it will request data asynchronously. So, set up a listener, listen to the current page number, as long as there’s a variation a request is made. The problem: If you click quite quickly, running through 1 to 5 in one go, that would give you 5 requests, so which page will finally show? The 5th page? That presumes that the ajax response for the 5th page request is the last to come in, but is that really the case? Not necessarily. So, this could cause confusion. Another problem, if you quickly click on the paging numbers 5 times consecutively, essentially you’re not interested in the content of the first 4 pages, so are not the first 4 requests of those pages all a waste of bandwidth? That’s not good either.

So the official answer to this was to design a solution: the listener side effect function can take an onInvalidate function as an argument, which can be registered to clear callbacks when they become invalid. This invalidation callback is triggered when the following situations occur:

- When the side effect is about to be re-executed.

- The listener has been stopped (if watchEffect has been used in the setup() or in a life cycle hook function, then it happens when the component is being unloaded).

```
watchEffect(onInvalidate => {
  const token = performAsyncOperation(id.value)
  onInvalidate(() => {
    // id has changed or watcher is stopped.
    // invalidate previously pending async operation
    token.cancel()
  })
})
```

First off, async operations need to be abortive ones. For timers, stopping them is pretty easy, clearInterval like functions should do. But for ajax, that requires using the abort ajax method provided by the ajax library (such as axios) to abort ajax.

To demonstrate abortive async operations, Here’s I’m going to write a directly executable example:
First, build a minimalist Node server on port 3000:

```
const http = require('http');
const server = http.createServer((req, res) => {
  res.setHeader('Access-Control-Allow-Origin', "*");
  res.setHeader('Access-Control-Allow-Credentials', true);
  res.setHeader('Access-Control-Allow-Methods', 'POST, GET, PUT, DELETE, OPTIONS');
  res.writeHead(200, { 'Content-Type': 'application/json'});
});
server.listen(3000, () => {
  console.log('Server is running...');
});
server.on('request', (req, res) => {
  setTimeout(() => {
    if (/\d.json/.test(req.url)) {
      const data = {
        content: 'I am the returned content from' + req.url
      }
      res.end(JSON.stringify(data));
    }
  }, Math.random() * 3000);
});
```

Next, you’ll need a Vue component that uses the server:

```
<template>
  <div>
    <div>content: {{ content }}</div>
    <button @click="changePageNumber">Page {{ pageNumber }}</button>
  </div>
</template>

<script>
import axios from 'axios';
import { ref, watchEffect } from 'vue';
export default {
  setup() {
    let pageNumber = ref(1);
    let content = ref('');
    const changePageNumber = () => {
      pageNumber.value++;
    }
    watchEffect((onInvalidate) => {
      const CancelToken = axios.CancelToken;
      const source = CancelToken.source();
      onInvalidate(() => {
        source.cancel();
      });
      axios.get(`http://localhost:3000/${pageNumber.value}.json`, {
          // cancelToken: source.token,
      }).then((response) => {
        content.value = response.data.content;
      }).catch(function (err) {
        if (axios.isCancel(err)) {
          console.log('Request canceled', err.message);
        }
      });
    });
    return {
      pageNumber,
      content,
      changePageNumber,
    };
  },
};
</script>
```

There are two possible outcomes for the above requests:

One is that the response is too fast for the request to be canceled, in that case, the request will return a 200 status code. But since its response is too fast, and no subsequent ajax has the chance to cancel it, means it has already finished before any subsequent requests start. Therefore, its content will definitely be overlapped by some of the following requests. So the content of these sort of requests will show momentarily, and then be overlapped by the following requests. It will definitely not later than the following requests.

The other is those red requests that have been cancelled, because their responses are slow, so they are cancelled.

Therefore, the final result must be correct, and it also saves a lot of bandwidth, as well as reduces system expenditures.

**Timing of side effects refresh**

> Vue’s reactivity system caches side effect functions and refreshes them asynchronously, which prevents unnecessary repetitive calls caused by multiple state changes in the same tick.

The meaning of the same tick is that Vue’s internal mechanism will merge the request for view refresh into one tick according to the most scientific calculation rules. Each “tick” refreshes the view once, such as a=1; b=2; will only trigger one view refresh. The "Tick" in $nextTick refers to this.

For example, watchEffect is listening to two variables, count and count2. When I call countAdd, do you think the listener will be called twice?
Of course not, Vue will merge it into one execution. The code is as follows, console.log will only execute once:

```
<template>
  <div>
    <div>{{count}} {{count2}}</div>
    <button @click="countAdd">Increase</button>
  </div>
</template>

<script>
import { ref, watchEffect } from 'vue';
export default {
  setup() {
    let count = ref(0);
    let count2 = ref(10);
    const countAdd = () => {
      count.value++;
      count2.value++;
    }
    watchEffect(() => {
      console.log(count.value, count2.value)
    })
    return {
      count,
      count2,
      countAdd
    }
  }
}
</script>
```

> In the core’s specific implementation, the component’s update function is also a monitored side effect. When a user-defined side effect function enters the queue, it is executed before all component updates by default.

The so-called component update function is a built-in function of Vue used to update the DOM, which is also a side effect.

At this point, there is a question, by default, will Vue execute the component DOM update first, or will it execute the listener first?

```
<template>
  <div>
    <div id="value">{{count}}</div> 
    <button @click="countAdd">Increase</button>
  </div>
</template>
<script>
import { ref, watchEffect } from 'vue';
export default {
  setup() {
    let count = ref(0);
    const countAdd = () => {
      count.value++;
    }
    watchEffect(() => {
      console.log(count.value)
      console.log(document.querySelector('#value') && document.querySelector('#value').innerText)
    })
    return {
      count,
      countAdd
    }
  }
}
</script>

// result

// before click
// 0
// null

// first click
// 1
// 0

// second click
// 2
// 1
```

Why does innerText print null before you click the button?

Because the fact is that it executes the listener first by default, and then updates the DOM, so the DOM hasn't been generated yet, so it is naturally null.

After the first and second clicks, you find that document.querySelector('#value').innerText always gets the content of the DOM before the click.

This also shows that by default, Vue executes the listener first, so it takes the content of the last time, and then performs the component update.

Vue 2 actually also uses this mechanism. Vue 2 uses this.$nextTick() to get the DOM after the component is updated.

In watchEffect, you don't need to use this.$nextTick() [and you can't], but there is a way to get the DOM after the component is updated, which is to use:

```
// it is triggered after the component is updated, so you can access the updated DOM.
// Note: This will also delay the initial run of the side effect until the component's first render is complete.
watchEffect(
  () => {
    /* ... */
  },
  {
    flush: 'post'
  }
)

// result

// before click
// 0
// 0

// first click
// 1
// 1

// second click
// 2
// 2
```

So the conclusion is, by default, watchEffect listener is executed first, followed by DOM update. If you want to operate on the "updated DOM", you need to configure flush: 'post'.

```
<template>
  <div>
    <div id="value">{{count}}</div> 
    <button @click="countAdd">Increase</button>
  </div>
</template>

<script>
import { ref, watchEffect } from 'vue';
export default {
  setup() {
    let count = ref(0);
    const countAdd = () => {
      count.value++;
    }
    watchEffect(() => {
      console.log(count.value)
      console.log(document.querySelector('#value') && document.querySelector('#value').innerText)
    },
    {
      flush: 'post'
    })
    return {
      count,
      countAdd
    }
  }
}
</script>
```

If you want to operate on the “updated DOM”, you need to set flush: 'post'.

The flush option has the following possible values:

- pre (default)

- post (triggered after the component is updated, so you can access the updated DOM. This also delays the start of the side effect until the first component rendering is complete.)

- sync (like watch, it forces the listener to trigger for each update, but this is inefficient and should rarely be needed.)

**Debugging the watchEffect listener**

The options onTrack and onTrigger can be used for debugging the listener's behavior.

onTrack is called when a reactive property or ref is tracked as a dependency.

onTrigger is called when a dependency change triggers a side effect.

Both callbacks will receive a debugger event with information about the dependency.

It is suggested to write debugger statements in the following callback to inspect dependencies:

```
watchEffect(
  () => {
    /* side effects */
  },
  {
    onTrigger(e) {
      debugger
    }
  }
)
```

onTrack and onTrigger only work in development mode.

## Conclusion

The special features about watch are that the watch listener function can add configuration options, or it can be configured as empty. When the configuration items are empty, the features of watch are as follows:

- **Lazy**: It won’t execute immediately at runtime

- **More specific**: You need to add the property under listening

- **Can access the value before the property is changed**: callback function will return the latest value and the value before the change

- **Can be configured**: The configuration items can make up for the shortcomings of watch features

**immediate**: Configure whether the watch property executes immediately. When the value is true, it will execute immediately as soon as it runs. When the value is false, it remains lazy.

**deep**: Configure whether watch is a deep listener. When the value is true, you can listen to all properties of the object. When the value is false, keep it more specific, you must specify it on the specific property.

**Features of watchEffect:**

- **Non-lazy**: It will execute immediately once it runs.

- **More abstract**: You don’t need to specify specifically who to listen to when using it, just use it directly in the callback function.

- **Can’t access the previous value**: You can only access the current latest value, and not the value before the modification.

Comparison between Vue 3 watch and Vue 2 watch.

The basic usage of Vue 3 watch and the instance method vm.$watch (or this.$watch) of Vue 2 is almost the same, but most programmers use the watch configuration item, so they may not be familiar with the $watch instance method. One advantage of the instance method is its flexibility, the first argument can accept a function, just like accepting a getter function

```
<template>
  <div>
    <button @click="r++">{{ r }}</button>
  </div>
</template>

<script>
import { ref, watch } from 'vue';
export default {
  setup() {
    let r = ref(1);
    let s = ref(10);
    watch(
      () => r.value + s.value,
      (newVal, oldVal) => {
        console.log(newVal, oldVal);
      }
    );
    return {
      r,
      s,
    };
  },
};
</script>
```

Vue 3 watch adds the ability to listen to multiple variables at the same time, using an array to represent the variables to be listened to. The callback parameters are of this structure: [newR, newS, newT], [oldR, oldS, oldT], don't misunderstand it as other wrong structures

```
<template>
  <div>
    <button @click="r++">{{ r }}</button>
  </div>
</template>

<script>
import { ref, watch } from 'vue';
export default {
  setup() {
    let r = ref(1);
    let s = ref(10);
    let t = ref(100);
    watch(
      [r, s, t],
      ([newR, newS, newT], [oldR, oldS, oldT]) => {
        console.log([newR, newS, newT], [oldR, oldS, oldT]);
      }
    );
    return {
      r,
    };
  },
};
</script>
```

The variable being listened to must be: A watch source can only be a getter/effect function, a ref, a reactive object, or an array of these types. In other words, it can be a getter/effect function, ref, Proxy and their array. It absolutely cannot be a plain object or basic data.

Does Vue 3 still have deep listening? Of course, it is by default, no need to declare. Of course, the premise is that the deep property is also reactive. If the deep property is not reactive, even if you write { deep: true }, it is useless.
