---
title: 'Understanding nextTick in Vue3'
date: 2024-06-18T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*bw3XTqb8qDgc4xdJ0DR75g.png"
categories: ["Code","Javascript"]
tags: ["Code","Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "We all use nextTick, and we all know that the role of nextTick is to execute delayed callbacks after the next DOM update cycle ends, allowing us to obtain updated DOM-related information."
---

We all use `nextTick`, and we all know that the role of `nextTick` is to execute delayed callbacks after the next DOM update cycle ends, allowing us to obtain updated DOM-related information. Before delving into the implementation principle of `nextTick`, let’s briefly review the execution mechanism of JS, as it is closely related to the implementation of `nextTick`.

## JS Execution Mechanism

We all know that JS is single-threaded, meaning it can only do one thing at a time, i.e., synchronously. This means all tasks need to queue up, and the tasks behind have to wait until the tasks in front are completed. If the tasks in front take too long, the subsequent tasks will have to wait continuously, which significantly affects the user experience. Therefore, the concept of asynchronous was introduced.

- **Synchronous tasks**: Tasks that are queued up and executed one by one on the main thread.

- **Asynchronous tasks**: Tasks that don’t enter the main thread but go into the task queue, further divided into macro-tasks and micro-tasks.

- **Macro-tasks**: Rendering events, requests, script, setTimeout, setInterval, setImmediate in Node, etc.

- **Micro-tasks**: Promise.then, MutationObserver (for monitoring DOM), Process.nextTick in Node, etc.

After the synchronous tasks in the execution stack are completed, it will take a macro-task from the task queue into the execution stack for execution. After completing all micro-tasks within that macro-task, it will take another macro-task from the queue, i.e., a macro-task, all micro-tasks, rendering, a macro-task, all micro-tasks, rendering… (not after all micro-tasks there will be rendering), thus forming a loop, which is the Event Loop.

`nextTick` is to create an asynchronous task, so it naturally has to wait until the synchronous tasks are completed before it is executed.

For more detailed content, you can refer to this article — [Did you know the running mechanism of JS?](https://medium.com/stackademic/thoroughly-combing-through-the-process-from-multi-process-browsing-to-js-single-threading-and-the-1125afab62ee)

## Usage

The first method: Pass in a callback function and manipulate the DOM within this function.

```
nextTick(() => {
     // The DOM manipulation to be performed
 });
```

The second method: Use async and await. Everything after await nextTick() is asynchronous code.

```const test = async () => {
    ......
    // Synchronous code
    await nextTick();
    // Asynchronous code, the DOM operation to be performed
    ......
};
```

## Usage Scenarios

- **Operation after DOM Update**: After data updates, the DOM needs to re-render. At this point, you can use nextTick to ensure that certain operations are performed after the DOM updates.

- **Callback after Component Rendering**: Some operations need to be performed after a component is fully rendered, such as accessing the component’s DOM elements.

- **Handling Asynchronous Update Issues**: In asynchronous operations, ensure callbacks are executed at the correct time.

## Source Code Analysis

Source Code File: `core-main/packages/runtime-core/src/scheduler.ts`

```
const resolvedPromise: Promise<any> = Promise.resolve()
let currentFlushPromise: Promise<void> | null = null

export function nextTick<T = void>(this: T, fn?: (this: T) => void): Promise<void> {
  const p = currentFlushPromise || resolvedPromise
  return fn ? p.then(this ? fn.bind(this) : fn) : p
}
```

It can be seen that nextTick accepts a function as a parameter and creates a microtask.

When we call `nextTick` on our page, it will execute this function, assigning our parameter fn to `p.then(fn)`, and after the tasks in the queue are completed, fn gets executed.

Due to the addition of a few methods for maintaining the queue, the sequence of execution is as follows:

`queueJob` -> `queueFlush` -> `flushJobs` -> fn of the `nextTick` parameter.

This content involves Vue’s task scheduling system. Vue3’s scheduling system contains three types of queues: `Pre queue`, `queue`, and `Post queue`. The `Pre queue` is used to manage tasks before the component’s DOM update, the `queue` is for managing tasks of the component’s DOM update, and the `Post queue` is for tasks after the DOM update. The `Pre queue` and `queue` share the same list, distinguished by the pre property. There are no priority distinctions in the `Pre queue` tasks, which are executed on a first-come, first-served basis. The `queue` and `Post queues` have a priority mechanism and can even allow cutting in line. The unique identifier id in the tasks also represents the task’s priority, where a smaller id indicates a higher priority. This is actually to ensure that tasks of parent components are executed before those of child components.

The implementation of Vue’s watch, component self-update, lifecycle, and other APIs all depend on its scheduling system.

The essence of Vue3’s scheduling system is the utilization of microtasks and the queue data structure. Tasks are placed in a queue, then unique ids are added to tasks for prioritization, ensuring that tasks in the queue are executed in the correct order. It also leverages the array’s includes and the Set data structure to eliminate duplicates, avoiding repeated execution of tasks and ensuring the performance of the Vue framework.

The essence of nextTick also utilizes the asynchronous execution mechanism of microtasks. Through a Promise.then callback, the nextTick callback is placed in the next event loop to access the updated DOM. It also leverages the bind method (function currying) to bind the component instance to nextTick in advance, simplifying nextTick’s parameters. Users don’t need to manually pass the component instance to nextTick to use it.
