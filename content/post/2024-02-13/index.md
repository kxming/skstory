---
title: 'Vue3 Development Solutions Two'
date: 2024-02-13T16:20:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*DYdGMu4svwz6UJ6Izma43g.png"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: "Vue3 Development Solutions2"
---

## Long list loading component

Mainly by monitoring whether the bottom DOM appears in the visible area, and then making data requests to handle some special situations. Usevue’s `useIntersectionObserver` api is used, which simply encapsulates the IntersectionObserver api, allowing us to more easily implement cross-monitoring of the visible area.

It mainly provides `isLoading` to display and load more dynamic icons, `isFinished` to determine whether the data request is completed, and load to request data props for the event.

```
<script setup>
import { useVModel, useIntersectionObserver } from '@vueuse/core'
import { onUnmounted, ref, watch } from 'vue'

const props = defineProps({
  isLoading: {
    type: Boolean,
    default: false
  },
  isFinished: {
    type: Boolean,
    default: false
  }
})

// Define loading binding event, load more event
const emits = defineEmits(['update:isLoading', 'load'])

const loading = useVModel(props, 'isLoading', emits)
// Load more
const loadingRef = ref(null)
// The first load, the visible area is true, and the visible area becomes false after the data is loaded
// If the visible area is not alternately visible, then the callback will not be executed

// Record whether it is at the bottom (whether it is intersecting)
const targetIsIntersecting = ref(false)
useIntersectionObserver(loadingRef, ([{ isIntersecting }]) => {
  // console.log(isIntersecting, props.isFinished, loading.value)
  targetIsIntersecting.value = isIntersecting
  emitLoad()
})

const emitLoad = () => {
  // When the bottom area appears, the data is not loaded, and when loading is false, request data
  if (targetIsIntersecting.value && !props.isFinished && !loading.value) {
    loading.value = true
    emits('load')
  }
}

/**
 * Handle the bug that the visible area judgement callback only executes once when the first data load is full screen
 *
 * Listen to the loading changes and trigger it again
 */
let timer = null
watch(loading, () => {
  // false => true（delay request data, wait for the previous request to finish, and then execute）=> false
  // Trigger load, delay processing, wait for rendering (although data returns, but the UI does not render, so targetIsIntersecting is still true) and useIntersectionObserver to trigger again
  // When one load can fill the container, when loading changes, do not let it load data. Because targetIsIntersecting is false. This delay time must be greater than the time after targetIsIntersecting changes

  // But for the case that one load cannot fill the container. targetIsIntersecting is always true, so you can load twice on the first screen. Wait for the next watch execution, just delay letting targetIsIntersecting change to false, then trigger emitLoad. At this time, it just stops the request.
  timer = setTimeout(() => {
    emitLoad()
  }, 500)
})

onUnmounted(() => {
  clearTimeout(timer)
})
</script>
```

There is a bug that is easy to occur here. When the amount of data we return is too small at one time, the bottom area is always within the but area. We will not be able to call the callback passed in useIntersectionObserver again, and we will not be able to request data again and load it. More.

So we need to monitor the changes in loading and trigger the data request again. But there is another problem. When we load too much data at once, we still request data multiple times. This is because although the data requested for the first time has come back, the interface has not yet been rendered. This means that the bottom area is still within the area, causing the data to be read again. ask. So we manually delay the request for data in the watch listener.

## Custom lazy loading instructions

We also need to use useVue’s useIntersectionObserver api. First, leave src blank. When entering the visible area, we assign src back.

```
import { useIntersectionObserver } from '@vueuse/core'

export default {
  mounted(el) {
    // Save image path
    const imgSrc = el.getAttribute('src')
    // Set image src to empty
    el.setAttribute('src', '')

    // Observe the visibility of the image
    const { stop } = useIntersectionObserver(el, ([{ isIntersecting }]) => {
      if(isIntersecting) {
        el.setAttribute('src', imgSrc)
        // Stop observing
        stop()
      }
    })
  }
}
```

Automatically register instructions through another method of vite’s Glob. Use `import.meta.globEager` to import all modules directly.

```
export default {
  install(app) {
    // Get all directive module objects.
    const modules = import.meta.globEager('./modules/*.js')
    for (let [key, value] of Object.entries(modules)) {
      const directiveName = key.replace('./modules/', '').split('.')[0]
      app.directive(directiveName, value.default)
    }
  }
}
```

## Confirm component

confirm Implementation ideas of components:

1. Create a confirm component

2. Create a function component and return a promise

3. At the same time, the h function is used to generate vnode of the confirm component.

4. Finally, use the render function to render vnode into body

After understanding the design idea of ​​the component, we need to analyze the props it should have.

```
const props = defineProps({
  title: {
    type: String
  },
  content: {
    type: String,
    required: true
  },
  // Button text
  cancelText: {
    type: String,
    default: 'Cancel'
  },
  confirmText: {
    type: String,
    default: 'Confirm'
  },
  // Trigger event when canceling and confirming, such as removing dom
  closeAfter: {
    type: Function
  },
  /**
   * Mainly to distinguish whether to click cancel or confirm
   */
  // Trigger event when confirming
  handleConfirmClick: {
    type: Function
  },
  // Trigger event when canceling
  handleCancelClick: {
    type: Function
  }
})
```

For the confirm component, we use a responsive data to control the animation of showing and hiding.

- When the pop-up box appears, we need to monitor the mounting moment and then control the display of the mask and pop-up box, otherwise the animation will fail.

- When we click to close the pop-up box, we cannot uninstall the component immediately, otherwise the animation will disappear immediately, so we delay the uninstallation.

```
// Animation duration, which is a state-driven dynamic CSS
const actionDuration = '0.5s'

// Control visibility of 'confirm'
const isVisible = ref(false)

// Display the dialog box as soon as the component is mounted. You can control component mounting and unmounting through function components.
// Use the 'mounted' life cycle hook to have an animation effect when it’s mounted.
onMounted(() => {
  isVisible.value = true
})

/**
 * Close the dialog box
 * The DOM is removed after the animation is completed using a timer.
 */
const handleClose = () => {
  // Animation appears only when it is hidden
  isVisible.value = false
  setTimeout(() => {
    // Unmount 'confirm' component
    props.closeAfter()
  }, parseInt(actionDuration.replace('0.', '').replace('s', '')) * 100)
}
```

The encapsulation of function components mainly uses h, render function operations.

- closeAfter : Mainly to uninstall the component when clicking anywhere to close the pop-up box.

- handleConfirmClick : Mainly when the confirmation button is clicked, the promise status is fulfilled, so that when the outside world uses the function component, the confirmed things can be operated in then.

- handleCancelClick : Mainly when the cancel button is clicked, the promise status is rejected, so that when the outside world uses the function component, the canceled things can be operated in the catch.

**The latter two functions are mainly to distinguish whether you clicked Cancel or Confirm.**

```
import { h, render } from 'vue'
import Confirm from './index.vue'

export default function createConfirm({
  title,
  content,
  cancelText = 'Cancel',
  confirmText = 'Confirm'
}) {
  return new Promise((resolve, reject) => { 

    /**
     * Remove 'confirm'
     */
    const closeAfter = () => {
      render(null, document.body)
    }

    /**
     * Callback when the 'confirm' button is clicked
     */
    const handleConfirmClick = resolve

    /**
     * Callback when the 'cancel' button is clicked
     */
    const handleCancelClick = reject

    // Create 'vnode' and pass in 'props'
    const vnode = h(Confirm, {
      title,
      content,
      cancelText,
      confirmText,
      closeAfter,
      handleConfirmClick,
      handleCancelClick
    })
    // Render component in 'body'
    render(vnode, document.body)
  })
}
```

## Message component

The implementation of the message component is very similar to confirm.

props need to specify the pop-up time and type

```
const props = defineProps({
  // message type
  type: {
    type: String,
    required: true,
    validate(val) {
      if (types.includes(val)) {
        return true
      } else {
        throw new Error('Please pass in a correct type value (error, warn, success)')
      }
    }
  },
  // message content
  content: {
    type: String,
    required: true
  },
  // message callback, unloads message after animation is completed
  closeAfter: {
    type: Function
  },
  // delay before deletion
  delay: {
    type: Number,
    default: 3000
  }
})
```

The main reason is that the hiding timing of the pop-up frame is different. In the message, it is hidden through the time control passed in from the outside world.

```
const isVisible = ref(false)

/**
 * To ensure animation is displayed when it appears, we need to show the corresponding content after the component is mounted.
 */
onMounted(() => {
  isVisible.value = true

  setTimeout(() => {
    isVisible.value = false
  }, props.delay)
})

// After the animation is complete, trigger the component to unmount using the 'after-leave' hook of the 'transition' component.
```

Function component implementation

```
import { h, render } from 'vue'
import Message from './index.vue'

export function createMessage({ type, content, delay = 3000 }) {

  /**
   * Callback when the animation is over
   */
  const closeAfter = () => {
    // message destruction
    render(null, document.body)
  }

  // Generate 'vnode'
  const vnode = h(Message, {
    type,
    content,
    delay,
    closeAfter
  })
  // Render component
  render(vnode, document.body)
}
```

## File download
File download related libraries:

- Small file download: [file-saver](https://github.com/eligrey/FileSaver.js)

- Large file download: [streamsaver](https://github.com/jimmywarting/StreamSaver.js)

Use the api directly and pass in the download path.

```
import { saveAs } from 'file-saver'

const handleDownload = (downloadPath) => {
  saveAs(downloadPath)
}
```

## Full screen display
We know that on native `dom` , some methods are provided for us to turn on or off full screen:

- `Element.requestFullscreen()`

- `Document.exitFullscreen()`

- `Document.fullscreen` Returns a Boolean value indicating whether the current document is in full-screen mode. Deprecated

- `Document.fullscreenElement` Returns the Element node that is being displayed in full-screen mode in the current document, or null if not.

**General browser**

Use `requestFullscreen()` and `exitFullscreen()` to achieve this

**Earlier version of Chrome browser**

Browsers based on the WebKit kernel need to add the webkit prefix, using `webkitRequestFullScreen()` and `webkitCancelFullScreen()` to achieve this.

**Earlier version of IE browser**

Browsers based on the Trident kernel need to add the ms prefix and use `msRequestFullscreen()` and `msExitFullscreen()` to achieve this. Note that the s in screen in the method is in lowercase.

**Earlier versions of Firefox**

Browsers based on the Gecko kernel need to add the moz prefix, using `mozRequestFullScreen()` and `mozCancelFullScreen()` to achieve this.

**Earlier version of Opera browser**

Opera browser needs to add the o prefix, use `oRequestFullScreen()` and `oCancelFullScreen()` to achieve this.

Considering compatibility, we can use the useFullscreen api provided by usevue

```
import { useFullscreen } from '@vueuse/core'

const imgRef = ref(null)
const { isFullscreen, enter, exit, toggle } = useFullscreen(imgRef)
const handleFullScreen = () => {
  imgRef.value.style.backgroundColor = 'transparent'
  enter()
}
```

## Function guide implementation

We can achieve this through the [driver.js](https://kamranahmed.info/driver.js/#) library.

Define the corresponding guidance steps.

```
export default [
  {
    // Which element to highlight
    element: '.guide-home',
    // Configuration object
    popover: {
      // Title
      title: 'Logo',
      // Description
      description: 'Click to return to the home page'
    }
  },
  {
    element: '.guide-search',
    popover: {
      title: 'Search',
      description: 'Search for the image you want'
    }
  },
  {
    element: '.guide-theme',
    popover: {
      title: 'Style',
      description: 'Choose a style you like',
      // Pop-up position
      position: 'left'
    }
  },
  {
    element: '.guide-my',
    popover: {
      title: 'Account',
      description: 'Your account information is marked here',
      position: 'left'
    }
  },
  {
    element: '.guide-start',
    popover: {
      title: 'Guide',
      description: 'You can view the guide information again here',
      position: 'left'
    }
  },
  {
    element: '.guide-feedback',
    popover: {
      title: 'Feedback',
      description: 'You can tell us about any dissatisfaction here',
      position: 'left'
    }
  }
]
```

Then call the api provided by the driver library

```
import Driver from 'driver.js'
import 'driver.js/dist/driver.min.css'
import steps from './steps'
import { onMounted } from 'vue'
/**
 * Guide page processing 
 */
let driver = null
onMounted(() => {
  driver = new Driver({
    // Prohibit closing by clicking on the screen mask
    allowClose: false,
    closeBtnText: 'Close',
    nextBtnText: 'Next',
    prevBtnText: 'Previous'
  })
})
  
/**
 * Start guiding
 */
const handleGuideClick = () => {
  // Define guide steps
  driver.defineSteps(steps)
  driver.start()
}
```

## Form validation
Third-party form validation library: [vee-validate](https://vee-validate.logaretm.com/v4/tutorials/dynamic-form-generator/).

This library provides three important components. Handle form components and form validation error prompts for us respectively.

```
import {
  Form as VeeForm,
  Field as VeeField,
  ErrorMessage as VeeErrorMessage
} from 'vee-validate'
```

Each form item can bind validation rules through rules props. The message corresponds to the name in the field.

```
<vee-form @submit="handleLogin">
    <vee-field
      class="dark:bg-zinc-800 dark:text-zinc-400 border-b-zinc-400 border-b-[1px] w-full outline-0 pb-1 px-1 text-base focus:border-b-main dark:focus:border-b-zinc-200 xl:dark:bg-zinc-900"
      name="username"
      :rules="validateUsername"
      type="text"
      placeholder="username"
      autocomplete="on"
      v-model="loginForm.username"
    />
    <vee-error-message
      class="text-sm text-red-600 block mt-0.5 text-left"
      name="username"
    >
</vee-form>
```

**What needs to be noted is: the verification function, true means that the form verification has passed, String means that the form verification has not passed, and the prompt text is given.**

```
/**
 * Form validation for Username
 */
export const validateUsername = (value) => {
  if (!value) {
    return 'Username is required'
  }

  if (value.length < 3 || value.length > 12) {
    return 'Username should be between 3-12 characters'
  }
  return true
}
```

For those that need to rely on other form values ​​for associated verification, we need to define rules through defineRule . For example: Confirm password input box verification.

```
/**
 * Form validation for Confirm Password
 *
 * Parameter two: An array representing the related form values
 */
export const validateConfirmPassword = (value, password) => {
  if (value !== password[0]) {
    return 'The two password entries must be consistent'
  }
  return true
}

/**
 * Define associated rules, for example confirm password
 */
defineRule('validateConfirmPassword', validateConfirmPassword)
```

ruleRule rules="validateConfirmPassword:@password"

```
<!-- Password -->
        <vee-field
          class="dark:bg-zinc-800 dark:text-zinc-400 border-b-zinc-400 border-b-[1px] w-full outline-0 pb-1 px-1 text-base focus:border-b-main dark:focus:border-b-zinc-200 xl:dark:bg-zinc-900"
          name="password"
          type="password"
          placeholder="Password"
          autocomplete="on"
          :rules="validatePassword"
          v-model="regForm.password"
        />
        <vee-error-message
          class="text-sm text-red-600 block mt-0.5 text-left"
          name="password"
        >
        </vee-error-message>
        <!-- Confirm Password -->
        <vee-field
          class="dark:bg-zinc-800 dark:text-zinc-400 border-b-zinc-400 border-b-[1px] w-full outline-0 pb-1 px-1 text-base focus:border-b-main dark:focus:border-b-zinc-200 xl:dark:bg-zinc-900"
          name="confirmPassword"
          type="password"
          placeholder="Confirm password"
          autocomplete="on"
          rules="validateConfirmPassword:@password"
          v-model="regForm.confirmPassword"
        />
        <vee-error-message
          class="text-sm text-red-600 block mt-0.5 text-left"
          name="confirmPassword"
        >
        </vee-error-message>
```

## Image cropping

To learn image cropping, we need to get an image and display it. How do we preview the image when we click to upload it? Let’s briefly introduce it.

**Picture Preview**

- The `URL.createObjectURL()` static method creates a DOMString containing a URL representing the object given in the parameter. The lifetime of this URL is bound to the document in the window that created it. This new URL object represents the specified File object or Blob object. Through `URL.createObjectURL(blob)` you can get a memory URL of the current file.

- `FileReader.readAsDataURL(file)` , a string of data:base64 can be obtained through `FileReader.readAsDataURL(file)` .

Execution time:

- `createObjectURL` is executed synchronously (immediately)

- `FileReader.readAsDataURL` is executed asynchronously (after a period of time)

Memory usage:

- `createObjectURL` returns a section of url with hash and is stored in memory until document triggers unload event (for example: document close ) or execute revokeObjectURL to release.

- `FileReader.readAsDataURL` returns base64 containing many characters and consumes more memory than blob url , but it will be automatically cleared from memory (through garbage) when not in use. Recycling mechanism) In terms of compatibility, both properties are compatible with browsers above IE10.

Compare the pros and cons:

Using createObjectURL can save performance and be faster, but you need to manually release the memory when not in use. If you don't care about device performance issues and want to get base64 of the image, it is recommended. Use `FileReader.readAsDataURL` .

[cropperjs](https://github.com/fengyuanchen/cropperjs) library to cut pictures

cropperjs is a very powerful image cropping tool, which can be applied to: native js, vue, react, etc. And the operation is very simple, and you can complete the picture cropping work in just a few simple steps.

```
import Cropper from 'cropperjs'
import 'cropperjs/dist/cropper.css'

// Mobile configuration object
const mobileOptions = {
  // Limit crop box within the size of canvas
  viewMode: 1,
  // Move canvas, crop box does not move
  dragMode: 'move',
  // Fixed aspect ratio of the crop box: 1:1
  aspectRatio: 1,
  // Crop box cannot be moved
  cropBoxMovable: false,
  // Crop box size cannot be adjusted
  cropBoxResizable: false
}

// PC configuration object
const pcOptions = {
  // Fixed aspect ratio of the crop box: 1:1
  aspectRatio: 1
}

/**
 * Image cropping processing
 */
const imageRef = ref(null)
let cropper = null
onMounted(() => {
  /**
   * Takes two parameters:
   * 1. DOM of the image to be cropped
   * 2. options configuration object
   */
  cropper = new Cropper(
    imageRef.value,
    isMobileTerminal.value ? mobileOptions : pcOptions
  )
})
```

Then we can get the cropped file object through cropper.getCroppedCanvas().toBlob .

```
// Get the cropped image
cropper.getCroppedCanvas().toBlob((blob) => {
  // The blob object after cropping
  console.log(blob)
})
```

## Make the h5 page jump as smooth as the native app page jump

Under normal circumstances, when we switch routes on the mobile terminal, in order to make the h5 page jump comparable to the native app, we will use the excessive animation provided by vue.

The main implementation logic is to first define the animation for entering and leaving the page, and dynamically change the transition animation name through routing jumps. Dynamically change the components of the cache component stack when jumping to achieve the component switching cache effect.

```
<template>
  <!-- Router outlet -->
  <router-view v-slot="{ Component }">
    <!-- Animation component -->
    <transition
      :name="transitionName"
      @before-enter="beforeEnter"
      @after-leave="afterLeave"
    >
      <!-- Cache component -->
      <!-- :key="$route.fullPath" to prevent caching when switching between dynamic routes -->
      <keep-alive :include="virtualTaskStack">
        <component
          :class="{ 'fixed top-0 left-0 w-screen z-50': isAnimation }"
          :is="Component"
          :key="$route.fullPath"
        />
      </keep-alive>
    </transition>
  </router-view>
</template>

<script>
const ROUTER_TYPE_NONE = 'none'
const ROUTER_TYPE_PUSH = 'push'
const ROUTER_TYPE_BACK = 'back'
</script>

<script setup>
import { ref, watch } from 'vue'
import { useRouter } from 'vue-router'

const props = defineProps({
  // Transition type. No transition animation for PC, set to 'none'
  routerType: {
    type: String,
    default: ROUTER_TYPE_NONE,
    validate(val) {
      if (
        val == ROUTER_TYPE_BACK ||
        val == ROUTER_TYPE_NONE ||
        val == ROUTER_TYPE_PUSH
      ) {
        return true
      } else {
        console.error(
          `Please pass in one of the types ${ROUTER_TYPE_NONE}, ${ROUTER_TYPE_BACK}, ${ROUTER_TYPE_PUSH}`
        )
        return false
      }
    }
  },
  // Main component name, corresponding to the first component in the task stack
  mainComponentName: {
    type: String,
    required: true
  }
})
// Cached components
const virtualTaskStack = ref([props.mainComponentName])

/**
 * Watch the transition type, then determine the animation name
 */
const transitionName = ref('')
watch(
  () => props.routerType,
  (val) => {
    transitionName.value = val
  }
)

/**
 * Change the cached components array every time the route changes.
 */
const router = useRouter()
router.beforeEach((to, from) => {
  // // Define current animation name
  // transitionName.value = props.routerType

  if (props.routerType === ROUTER_TYPE_PUSH) {
    // Push to stack
    virtualTaskStack.value.push(to.name)
  } else if (props.routerType === ROUTER_TYPE_BACK) {
    // Pop from stack
    virtualTaskStack.value.pop()
  }

  // Default to clear the stack when entering the main page
  if (to.name === props.mainComponentName) {
    clearTask()
  }
})

/**
 * Animation starts
 */
const isAnimation = ref(false)
const beforeEnter = () => {
  isAnimation.value = true
}

/**
 * Animation ends
 */
const afterLeave = () => {
  isAnimation.value = false
}

/**
 * Clear stack
 */
const clearTask = () => {
  virtualTaskStack.value = [props.mainComponentName]
}
</script>

<style lang="scss" scoped>
// When pushing a new page: entering animation for the new page
.push-enter-active {
  animation-name: push-in;
  animation-duration: 0.6s;
}
// When pushing a new page: leaving animation for the old page
.push-leave-active {
  animation-name: push-out;
  animation-duration: 0.6s;
}
// Animation for new page entering when pushing a new page
@keyframes push-in {
  0% {
    transform: translate(100%, 0);
  }
  100% {
    transform: translate(0, 0);
  }
}
// Animation for old page exiting when pushing a new page
@keyframes push-out {
  0% {
    transform: translate(0, 0);
  }
  // The old page only moves 50% of the animation here, but the new page moves 100%, so it will be squeezed out.
  100% {
    transform: translate(-50%, 0);
  }
}

// Animation for the page about to be displayed when moving backwards
.back-enter-active {
  animation-name: back-in;
  animation-duration: 0.6s;
}
// Animation for the page moving backwards
.back-leave-active {
  animation-name: back-out;
  animation-duration: 0.6s;
}
// Animation for the page about to be displayed when moving backwards
@keyframes back-in {
  0% {
    width: 100%;
    transform: translate(-100%, 0);
  }
  100% {
    width: 100%;
    transform: translate(0, 0);
  }
}
// Animation for page moving backwards
@keyframes back-out {
  0% {
    width: 100%;
    transform: translate(0, 0);
  }
  100% {
    width: 100%;
    transform: translate(50%, 0);
  }
}
</style>
```
