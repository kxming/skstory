---
title: 'Writing a More Stable Timer with Web Worker'
date: 2024-06-11T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*xE1QhWRUKI-g2MKZs_6reg.jpeg"
categories: ["Code","Javascript"]
tags: ["Code","Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "When the browser is minimized or in the background, certain optimizations are applied to setTimeout and setInterval. This might involve throttling..."
---

When the browser is minimized or in the background, certain optimizations are applied to setTimeout and setInterval. This might involve throttling, or the tasks may be consolidated until the browser comes back to the foreground (at which point, you might notice multiple timer callbacks being triggered in a short period). Therefore, if you need a more stable timer, such as one that doesn’t pause for the logic in your program, you can make use of a Web Worker.

The principle is quite straightforward, as Web Workers run on a separate thread and are not affected by the main thread. Example:

```
let normalTime = 0;
setInterval(() => {
  console.debug('Normal: ', normalTime++);
}, 1000);

const blob = new Blob(
  [
    `let workerTime = 0;
    self.setInterval(()=>{
      console.debug('In Worker: ', workerTime);
      self.postMessage(workerTime);

      workerTime++;
    }, 1000);`
  ],
  { type: 'application/javascript' }
  );
const worker = new Worker(URL.createObjectURL(blob));
worker.onmessage = ev => {
  console.debug('Out of Worker: ', ev.data);
};
```

Using a Blob, a Web Worker is dynamically created, within which a timer is started to send out time messages.

Running this code and then minimizing the browser, after a while, the value of `normalTime` will be less than `workerTime`. `workerTime` represents a more accurate count value. Although it is limited by the inherent accuracy of `setInterval` and is not precisely timed, this method is more stable.
