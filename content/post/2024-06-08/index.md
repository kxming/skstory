---
title: 'Encapsulation of Vue3 Event Tracking Directive'
date: 2024-06-08T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*OorOgEjYFEubQx8_zQ920A.png"
categories: ["Code","Javascript"]
tags: ["Code","Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "Event tracking analysis is a common data collection method used in website analysis."
---

## What is Event Tracking?

Event tracking analysis is a common data collection method used in website analysis.

Simply put, the main purpose of front-end event tracking is for operation and development personnel to collect user behavior data and page performance data for subsequent data analysis. For example, obtaining the page load time under various networks, or the user’s stay time on a certain page!

## What is the purpose of event tracking?

In today’s era where the customer is king and the internet competition is fierce, customizing different content for each user’s preferences and deciding the product iteration direction according to user preferences has become an essential focus for internet companies. Therefore, event tracking have become an indispensable way to obtain information. So, what are the purposes of event tracking? What data do we need to collect?

## Performance Monitoring
In small projects, as the number of users is not large, people think it’s okay as long as it’s passable. However, when the number of users surges, performance monitoring becomes very important. Because it allows you to know some potential problems and bugs, and iterate quickly to obtain a better user experience! Generally, we need to pay attention to a few points during performance monitoring:

1. White screen duration

2. HTTP request time of important pages

3. Rendering time of important pages

4. First screen loading duration

Some might ask, isn’t white screen duration the same as the first screen loading duration? Here, the white screen duration actually refers to the time from the request to the appearance of the UI skeleton on the page (this tests the time from the request domain name to the DNS resolution completion, returning the page skeleton), whereas the first screen loading duration refers to the time all dynamic content on the page is loaded, including the time from AJAX data to rendering on the page.

## Data Monitoring
Data monitoring means being able to obtain user behavior, and we need to pay attention to a few points:

1. PV (Page View)

2. UV (Unique Visitor)

3. Record operating systems and browsers

4. Record user’s stay time on the page

5. Source webpage of entry to the current page (i.e., where the conversion came from)

## Encapsulating Event Tracking Directives in Vue3

`track.ts` file

```
import HttpAxios from '@/utils/axios'
​
// Define the burying point request
export function track(params) {
  return HttpAxios.post(
    'http://xxxxxx.xxxxx',
    params,
    {
      isCloseLoading: true
    }
  )
}
​
export default {
  install(Vue, options) {
    options = options || {}
    /**
     * Format the data bound to the DOM
     * @param {*} binding
     */
    function formatBinding(binding) {
      let trackData = ''
      let eventMode = 'click'
      if (typeof binding.value === 'object') {
        if ('event' in binding.value) {
          eventMode = binding.value.event
        }
        if ('data' in binding.value) {
          trackData = binding.value.data
        } else {
          trackData = binding.value
        }
      } else {
        trackData = binding.value
      }
      return {
        eventMode,
        trackData
      }
    }
    // Initialize
    if ('init' in options && options.init instanceof Function) {
      try {
        options.init()
      } catch (error) {
        if (options.debug) {
          console.log(error)
        }
      }
    }
    Vue.directive('track', {
      mounted(el, binding) {
        const format = formatBinding(binding)
        el.trackData = format.trackData
        const params = {
          systemName: options.systemName,
          ...el.trackData // Data bound by the directive
        }
        el.addEventListener(format.eventMode, e => {
          try {
            if ('callback' in options && options.callback instanceof Function) {
              options.callback(params)
            } else {
              // If no callback function is defined, call the track method by default
              track(params)
            }
            if (options.debug) {
              console.log(el.trackData)
            }
          } catch (error) {
            if (options.debug) {
              console.log(error)
            }
          }
        })
      },
      update(el, binding) {
        const format = formatBinding(binding)
        el.trackData = format.trackData
      }
    })
  }
}
```

Include in the `main.ts` file

```
// Import burying point
import VTrack from '@monorepo/shared/utils/track'
​
// Create Vue instance
const app = createApp(App)
​
// 1. Mount burying point without callback function
app.use(VTrack, { systemName: 'Basic Platform', debug: false })
​
// 2. Mount burying point with callback function
app.use(VTrack, {
  callback(data, e) {
    // Custom burying point request can be defined
    console.log(data, e);
  },
  systemName: 'Basic Platform',
  debug: false,
});
```

Usage:

```
<template>
  <button v-track="{ menuName: 'Button' }">DEBUG</button>
</template>
```
