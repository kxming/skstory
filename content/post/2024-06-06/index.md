---
title: 'How to control multiple concurrent requests?'
date: 2024-06-06T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*C7wa55tbYeJsoU9Ay8BbRg.jpeg"
categories: ["Code","Javascript"]
tags: ["Code","Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "Suppose there are 100 requests to be sent, design an algorithm to control concurrent requests (with a maximum concurrency of 10) using Promise, to complete all 100 requests."
---

First, let’s simulate the 100 requests:

```
// Request List
const requestList = [];

for (let i = 1; i <= 100; i++) {
  requestList.push(
    () =>
      new Promise(resolve => {
        setTimeout(() => {
          console.log('done', i);
          resolve(i);
        }, Math.random() * 1000);
      }),
  );
}
```

## Promise.all()

Upon first encountering this problem, most people would likely think of `Promise.all`, as it is the most common way to handle concurrent requests. Let’s implement it:

```
const parallelRun = async max => {
  const requestSliceList = [];
  for (let i = 0; i < requestList.length; i += max) {
    requestSliceList.push(requestList.slice(i, i + max));
  }

  for (let i = 0; i < requestSliceList.length; i++) {
    const group = requestSliceList[i];
    try {
      const res = await Promise.all(group.map(fn => fn()));
      console.log('Response:', res);
    } catch (error) {
      console.error(error);
    }
  }
};
```

The result:

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*LCXXm4fQqiX41fnDuOl1jQ.gif)

Each time, 10 requests are sent concurrently. Once these 10 requests are completed, the next 10 requests are sent, perfectly fulfilling the requirement.

However, a new question arises: what happens if one of these requests fails?

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*v1Mywb8-45hHXLw2rX2z2g.gif)

If there’s a failure in one request, the entire `Promise.all` fails without returning any value. If one request in a group fails, it becomes impossible to retrieve the return values of the other members in that group. While this may be acceptable in scenarios where return values aren’t necessary, in actual business practices, return values are crucial data.

We can tolerate the lack of a return value from a failed interface, but it’s unacceptable for a failure in one request to result in no return values for the other 9 requests in the same group.

Given that a failed request interrupts `Promise.all`, is there a method to avoid being disrupted by failures?

Indeed, there is — it’s called `Promise.allSettled`!

## Promise.allSettled()

Let’s take a look at the introduction from MDN.

> The `Promise.allSettled()` method is one of the promise concurrency methods. `Promise.allSettled()` is typically used when you have multiple asynchronous tasks that are not dependent on one another to complete successfully, or you'd always like to know the result of each promise.

Replace `Promise.all()` with `Promise.allSettled()`:

```
const parallelRun = async max => {
  const requestSliceList = [];
  for (let i = 0; i < requestList.length; i += max) {
    requestSliceList.push(requestList.slice(i, i + max));
  }

  for (let i = 0; i < requestSliceList.length; i++) {
    const group = requestSliceList[i];
    try {
      // Replace
      const res = await Promise.allSettled(group.map(fn => fn()));
      console.log('Response: ', res);
    } catch (error) {
      console.error(error);
    }
  }
};
```

The result:

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*BBTMDkKPMIoetXqB4_4NSg.png)

As can be seen, when all interfaces return values normally, the return values will accurately record whether the current request succeeded or failed.

New question: If one request is very time-consuming, the response of that group of requests will be slow, blocking subsequent concurrent interface calls.

**Analyse**:

Using `Promise.all()` or `Promise.allSettled()` to concurrently handle 10 requests at a time can indeed meet concurrency requirements, but the efficiency is low. If there is one or more slow interfaces, two problems will occur:

1. The return of the concurrent group with the slow interface will be very slow; one slow interface delays the other nine, which is counterproductive.

2. Although we are capable of handling 10 concurrent requests, a slow interface causes the other nine concurrent slots in that group to be wasted, which cruelly extends the concurrency time for these 100 interfaces.

3. The subsequent concurrent groups behind the slow interface group are all blocked, making it even slower.

**Solution**:

A running pool and a waiting queue can be maintained, always keeping 10 requests concurrently in the running pool.

When one request in the running pool is completed, a new request is taken from the waiting queue and placed into the running pool to run, thus ensuring the running pool is always operating at full capacity.

Even if there is a slow interface, it will not block subsequent interfaces from entering the pool.

```
// Running pool
const pool = new Set();

// Waiting queue
const waitQueue = [];

/**
 * @description: Limit the number of concurrent requests
 * @param {*} reqFn: Request method
 * @param {*} max: Maximum concurrency
 */
const request = (reqFn, max) => {
  return new Promise((resolve, reject) => {
    // Check if the running pool is full
    const isFull = pool.size >= max;

    // Wrapped new request
    const newReqFn = () => {
      reqFn()
        .then(res => {
          resolve(res);
        })
        .catch(err => {
          reject(err);
        })
        .finally(() => {
          // After the request is completed, remove it from the running pool
          pool.delete(newReqFn);
          // Take out a new request from the waiting queue and put it into the running pool for execution
          const next = waitQueue.shift();
          if (next) {
            pool.add(next);
            next();
          }
        });
    };

    if (isFull) {
      // If the running pool is full, put the new request into the waiting queue
      waitQueue.push(newReqFn);
    } else {
      // If the running pool is not full, add a new request to the pool and execute it
      pool.add(newReqFn);
      newReqFn();
    }
  });
};

requestList.forEach(async item => {
  const res = await request(item, 10);
  console.log(res);
});
```

The result:

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*1vE8yqXy4Zksy1rXN9R9xw.gif)

As can be seen, the 100 interfaces are executed continuously without any waiting or blocking.

## Summary

This article mainly summarizes methods to limit concurrency for 100 requests:

- `Promise.all()` is the simplest way to control concurrency, but if a request fails, that group will not return a value.

- `Promise.allSettled()` solves the issue of `Promise.all()`, but it suffers from slow interfaces blocking subsequent requests and wasting other concurrent slots.

- By maintaining a running pool, when a request in the pool completes, a new request is taken from the waiting queue and added to the pool for execution until all requests have entered the pool.
