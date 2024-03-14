---
title: '7 Advanced Js async/await Usage Techniques'
date: 2024-02-19T16:22:38+08:00
draft: true
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*vsZ7fYFnKwjK2xhh2VpyEA@2x.png"
categories: ["Javascript"]
tags: ["Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "Asynchronous programming in JavaScript has evolved from callbacks to Promises, and now to the widely used async/await syntax..."
---

Asynchronous programming in JavaScript has evolved from callbacks to Promises, and now to the widely used async/await syntax. The latter not only simplifies asynchronous code but also brings it closer to the logic and structure of synchronous code, greatly enhancing the readability and maintainability of the code. After mastering the basic usage, here are some advanced usage to fully take advantage of async/await to implement more complex asynchronous process control.

---

## async/await and Higher-Order Functions

When you need to perform asynchronous operations on elements in an array, you can use async/await in conjunction with array higher-order functions (such as map, filter, etc.).

```
// Asynchronous filter function
async function asyncFilter(array, predicate) {
    const results = await Promise.all(array.map(predicate));
    return array.filter((_value, index) => results[index]);
}
// Example
async function isOddNumber(n) {
    await delay(100); // Simulate asynchronous operations
    return n % 2 !== 0;
}
async function filterOddNumbers(numbers) {
    return asyncFilter(numbers, isOddNumber);
}
filterOddNumbers([1, 2, 3, 4, 5]).then(console.log); // Output: [1, 3, 5]
```

## Control Concurrency

When dealing with scenarios such as file uploading, you may need to limit the number of concurrent asynchronous operations to prevent system resources from running out.

```
async function asyncPool(poolLimit, array, iteratorFn) {
    const result = [];
    const executing = [];
    for (const item of array) {
        const p = Promise.resolve().then(() => iteratorFn(item, array));
        result.push(p);
        if (poolLimit <= array.length) {
            const e = p.then(() => executing.splice(executing.indexOf(e), 1));
            executing.push(e);
            if (executing.length >= poolLimit) {
                await Promise.race(executing);
            }
        }
    }
    return Promise.all(result);
}
// Example
async function uploadFile(file) {
    // File upload logic
}
async function limitedFileUpload(files) {
    return asyncPool(3, files, uploadFile);
}
```

## Optimize recursion with async/await

Recursive functions are a common technique in programming, and async/await makes it easy for recursive functions to perform asynchronous operations.

```
// Asynchronous recursive function
async function asyncRecursiveSearch(nodes) {
    for (const node of nodes) {
        await asyncProcess(node);
        if (node.children) {
            await asyncRecursiveSearch(node.children);
        }
    }
}
// Example
async function asyncProcess(node) {
    // Asynchronous node processing logic
}
```

## Asynchronously Initialize Class Instances

In JavaScript, a class’s constructor cannot be asynchronous. But you can implement asynchronous initialization of class instances through the factory function pattern.

```
class Example {
    constructor(data) {
        this.data = data;
    }

    static async create() {
        const data = await fetchData(); // Asynchronously fetch data
        return new Example(data);
    }
}

// Usage
Example.create().then((exampleInstance) => {
    // Use an asynchronously initialized class instance
});
```

## Use await Chain Calls in async Functions

By using await, you can intuitively execute asynchronous operations in chain calls in order.

```
class ApiClient {
    constructor() {
        this.value = null;
    }
    async firstMethod() {
        this.value = await fetch('/first-url').then(r => r.json());
        return this;
    }
    async secondMethod() {
        this.value = await fetch('/second-url').then(r => r.json());
        return this;
    }
}
// Usage
const client = new ApiClient();
const result = await client.firstMethod().then(c => c.secondMethod());
```

## Combine async/await and Event Loop

By using async/await, you can better control the event loop, such as handling DOM events or timers.

```
// Asynchronous timer function
async function asyncSetTimeout(fn, ms) {
    await new Promise(resolve => setTimeout(resolve, ms));
    fn();
}
// Example
asyncSetTimeout(() => console.log('Timeout after 2 seconds'), 2000);
```

## Simplify Error Handling with async/await

Error handling is an important part of asynchronous programming. With async/await, error handling logic can be naturally integrated into synchronous code.

```
async function asyncOperation() {
    try {
        const result = await mightFailOperation();
        return result;
    } catch (error) {
        handleAsyncError(error);
    }
}
async function mightFailOperation() {
    // Asynchronous operation that might fail
}
function handleAsyncError(error) {
    // Error handling logic
}
```

With these seven advanced async/await usages, developers can handle complex asynchronous logic in JavaScript in a more declarative and intuitive way while keeping the code neat and maintainable. Continuously applying and mastering these usage in practice can effectively enhance programming efficiency and project quality.
