---
title: 'Vue3 Development Solutions One'
date: 2024-02-12T16:22:33+08:00
draft: true
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*DYdGMu4svwz6UJ6Izma43g.png"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: "Vue3 Development Solutions"
---

## Customized, high-availability front-end style processing solution

Pain points in CSS processing under enterprise-level projects：

- Unified variables are difficult to maintain

- Lots of className overhead

- The writing burden caused by the separation of HTML and CSS

- Responsive, theme switching is complex to implement

For more pain points, see [CSS Utility Classes and “Separation of Concerns”](https://adamwathan.me/css-utility-classes-and-separation-of-concerns/)

In response to the above problems, we can solve it through [tailwindcss](https://tailwindcss.com/). Let’s look at its specific usage below.

**Installation**

```
yarn add tailwindcss postcss autoprefixer -D
```

Initializes the `tailwindcss.config.js` configuration file and also creates the `postcss.config.js` file.

```
// -p This represents creating a basic configuration file.
npx tailwindcss init -p
```
```
// tailwindcss.config.js
/** @type {import('tailwindcss').Config} */
export default {
  // The scope of files applied by Tailwind CSS.
  content: ['./index.html', './src/**/*.{vue,js}'],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

Add instructions to load Tailwind to your CSS file

```
@tailwind base;
@tailwind components;
@tailwind utilities;
```

Use built-in class names on elements

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*PNyxfP_-GYnFZaNqLon0AQ.jpeg)

Tailwindcss is officially introduced to quickly build modern websites without leaving HTML. Specifically, tailwind provides many class names, all of which define specific css. You can quickly build a website by directly adding the corresponding class name when writing HTML.

**The design concept of tailwindcss**

First, let’s take a look at the css granular design form

- Inline styles. The highest degree of freedom and the strongest customization. But it is inconvenient to reuse styles.

```
<div style="color: red; font-size: 20px">zh-llm</div>
```

- Atomized css, each class name represents a type of css style. The degree of freedom is still very strong, the customization is also very high, and the styles can be reused. But a lot of meaningless class names will be written. Among them, tailwindcss is this design.

```
<div class="text-sky-400">zh-llm</div>
```

- In the traditional form, one or several semantic classes are used to describe a CSS attribute. It is encapsulated, has strong semantics, and has average freedom and customizability (most class names are written with the entire set of CSS attributes of the corresponding element). However, there are a large number of semantic classes, which require switching back and forth between HTML and CSS when writing.

```
<div class="container clear"></div>
```

- In component form, the structure and style are directly defined in the current component. It has strong encapsulation and strong semantics. However, the degree of freedom and customizability are relatively poor. And the style is fixed, which is more suitable for background projects. Such as element-plus and so on.

```
 <my-component />
```

- Comparing the four design methods, we can see that atomized CSS has a degree of freedom, is customizable, and has good reusability. It only has the disadvantage of writing a large number of meaningless class names. Compared with its advantages, the disadvantages can be ignored. But for those who maintain the project, if they don’t understand the class names defined in tailwindcss, it may be a very headache.

For high personalization, high interactivity, and highly customized front-end project style solutions, the atomized CSS form is more suitable.

When developing with vscode, we can install a Tailwind CSS IntelliSense plug-in to prompt for class names to help us develop better.

## VueUse A practical toolset for Vue’s compositional API

[VueUse](https://www.vueusejs.com/), a set of practical tools based on Vue’s composite API. It encapsulates many common and reusable functions. It supports Vue2, Vue3, and Nuxt. It can be understood as the lodash of the Vue world. It can be used out of the box and is very convenient.

The core package has 140+ combined functions, and also provides 10 extension plug-ins, totaling about 290+ functions.

**Function**

- State: Manage user status (such as: global, local storage, session storage)

- Elements: Element processing related (such as: element dragging, element visibility, window size)

- Browser: Browser related (such as: updating page title, media query, clipboard)

- Sensors: monitor different DOM events, input events, network events, etc.

- Network: Network request related

- Animation: Transitions, timeouts and timing functions

- Component: Provides abbreviations for different component methods

- Watch: Provides some monitors (such as: monitoring changes after promises, anti-shake monitoring, throttling monitoring)

- Reactivity: Responsive function related

- Array: Responsive array processing

- Time: Provides response time formatting function
- Utilities: general functions, such as throttling, anti-shake, etc.

- Plugin: Electron、Firebase、Head、Integrations、Math、Motion、Router、RxJS、SchemaOrg、Sound

The core functions are all in the packages/core folder.

**Installation**

```
  npm i @vueuse/core
  // or
  yarn add @vueuse/core
  // or
  pnpm add @vueuse/core 
```

**Use**

- all it directly as a function: for example, `const { toggle } = useFullScreen()`, which is also our most commonly used method.

- Used as a component: Some functions are provided, such as `<UseFullscreen><UseFullscreen>` .

When called directly as a function, you can receive responsive parameters. Most functions will return a refs object, and you can use object destructuring to obtain the required content. Functions usually start with use.

For component usage, in addition to installing the `@vueuse/core` core package, you also need to install `@vueuse/components`, that is

```
pnpm add @vueuse/core  @vueuse/components
```

VueUse encapsulates many practical functions, such as: clipboard (useClipboard), anti-shake (useDebounceFn), setting the title of the web page (useTitle), and listening for events (useEventListener). Page uninstallation can automatically help us delete event monitoring without us having to delete it manually. etc. Before encapsulating hooks yourself, you can go to the official website to see if they have been encapsulated to improve development efficiency.

VueUse also has good interactive documentation and function demonstrations. It is written in TS. The core source code of each function is relatively small and easy to read. In addition to using its encapsulated functions, we can also learn function encapsulation skills by looking at its source code. Learn good coding habits.

## Why is vite faster than webpack?

When building during webpack development, your entire application will be crawled and built by default before it can provide services. This will cause any error in your project (even if the current error is not a module referenced on the homepage), it will still affect it. to your entire project build. So the bigger your project is, the longer the build time will be and the slower the project startup will be.

Vite will not build your entire project at the beginning, but will divide the modules in the reference into two parts: dependency and source code (project code). For the source code part, it will split the code modules according to routing, and will only go Build what you have to build from the get-go.

At the same time, Vite provides the source code to the browser in the form of native ESM, allowing the browser to take over part of the packaging work.

## vite development configuration

**Configure path alias**

```
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import {join} from "path"

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      "@": join(__dirname, "/src")
    }
  }
})
```

**Development environment solves cross-domain issues**

```
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import {join} from "path"

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      "@": join(__dirname, "/src")
    }
  },
  server: {
    proxy: {
      // Proxy all /api requests
      "/api": {
        target: "target origin",
        // Change the origin of the request to the value of target
        changeOrigin: true,
      }
    }
  }
})
```

**Configure environment variables**

Enterprise-level projects will distinguish many environments for us to test and trial. We cannot let our test data contaminate the online data.
Therefore, Vite also provides a way to configure our environment files, allowing us to easily select the corresponding interface address and so on through some environments.

The format of `.env.[mode]` can load different content in different modules.

Environment loading priority:

- A file that specifies a pattern (e.g. `.env.production` ) will take precedence over a generic form (e.g. `.env` ).

- In addition, environment variables that already exist when Vite is executed have the highest priority and will not be overwritten by the `.env` class file. For example when running `VITE_SOME_KEY=123` vite build .

- `.env` The class file will be loaded when Vite starts, and changes will take effect after restarting the server.

We can obtain loaded environment variables starting with `VITE_` through `import.meta.env.*` in the code.

```
// .env.development
VITE_BASE_API = "/api"
// package.json
"scripts": {
    "dev": "VITE_BASE_API=/oop vite",
}
```

After executing yarn dev , we can find that import.meta.env.VITE_BASE_API is the parameter specified in the command line.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*DEyxvyBRfdg2uXD9dCXIkg.jpeg)

## Automatic registration of common components

[Vite’s Glob import](https://vitejs.dev/guide/features.html#glob-import) function: This function can help us import multiple modules in the file system

```
const modules = import.meta.glob('./dir/*.js')
// translate：
const modules = {
  './dir/foo.js': () => import('./dir/foo.js'),
  './dir/bar.js': () => import('./dir/bar.js')
}
```

Then introduce it through the way of registering asynchronous components provided by vue. Vue’s `defineAsyncComponent` method: This method can create an asynchronous component that is loaded on demand. Based on the above two methods, the component can be automatically registered.

```
// import SvgIcon from './svg-icon/index.vue'
// import HmPopup from './popup/index.vue'

import { defineAsyncComponent } from 'vue'

// const components = [SvgIcon, HmPopup]

export default {
  install(app) {
    // components.forEach((element) => {
    //   app.component(element.name, element)
    // })
    // Get all index.vue under the current path
    const components = import.meta.glob('./*/index.vue')
    // Traverse the obtained component module
    for (let [key, component] of Object.entries(components)) {
      const componentName = 'hm-' + key.replace('./', '').split('/')[0]
      // asynchronously import components under specified path using defineAsyncComponent
      app.component(componentName, defineAsyncComponent(component))
    }
  }
}
```

In fact, if the components all provide the name attribute, we can directly manually introduce each component module and then achieve semi-automatic registration.

The advantage of providing a name for a component is that it is convenient to find each component when debugging in vue-devtools.

In the vue official website, in versions 3.2.34 or above, the single-file component using `<script setup>` will automatically generate the corresponding name option based on the file name, even when used with `<KeepAlive>` There is no need to declare manually when using it. But for us developers whose file names are all index.vue, there is no way.

## Use svg as icon

First we need to encapsulate a general svg component to use svg icons.

```
<template>
  <svg aria-hidden="true">
    <use :xlink:href="symbolId" :fill="color" :fillClass="fillClass" />
  </svg>
</template>

<script setup>
import { computed } from 'vue'

const props = defineProps({
  name: {
    type: String,
    required: true
  },
  color: {
    type: String
  },
  fillClass: {
    type: String
  }
})

// Generate a unique icon id #icon-xxx
const symbolId = computed(() => `#icon-${props.name}`)
</script>
```

Then register the svg universal component globally. Here we use the plug-in method.

```
import SvgIcon from "./svg-icon/index.vue"

export default {
  install(app) {
    app.component("SvgIcon", SvgIcon)
  }
}
```

After registering directly through use in main.js, it can be used.

```
<svg-icon name="back"></svg-icon>
```

However, the path of the svg icon cannot be known in the project. We need to use the `vite-plugin-svg-icons` plug-in to specify the search path.

Configure svg related content in `vite.config.js`

```
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import {join} from "path"
import {createSvgIconsPlugin} from "vite-plugin-svg-icons"

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [
    vue(),
    createSvgIconsPlugin({
      // Specify the icon folders that need to be cached
      iconDirs: [join(__dirname, "/src/assets/icons")],
      // Specify symbolId format, which is the href used by svg.use
      symbolId: "icon-[name]"
    })
  ],
})
```

Import and register svg-icons in main.js, it will register all svg images in the specified folder on the homepage.

```
// register svg-icons
import "virtual:svg-icons-register"
```

## Persistent state data vuex-persistedstate

[vuex-persistedstate](https://github.com/robinvdvleuten/vuex-persistedstate), as a plug-in of vuex, can persist the data in the store to prevent data loss due to page refresh and other operations. (When running again, the cached data will be used as the initial value of the corresponding state attribute)

```
import { createStore } from "vuex";
import createPersistedState from "vuex-persistedstate";

const store = createStore({
  // ...
  plugins: [createPersistedState({
      key : 'categoryList', // The key for cache,
      paths: ['category'], // An array of any paths to partially persist the state. If no paths are given, the complete state is persisted. If an empty array is given, no state will be persisted. The path must be specified using dot notation. If using modules, include the module name. Such as: "auth.user". Default is undefined.
  })],
});
```

## Theme switching implementation

Implementation idea: (This solution is based on the tailwindcss plug-in)

The `tailwind.config.js` configuration file needs to be added

```
darkMode: 'class'
```

Store current theme type in vuex

```
// current theme
import { THEME_LIGHT } from '@/constants'
export default {
  namespaced: true,
  state: () => ({
    themeType: THEME_LIGHT
  }),
  mutations: {
    setThemeType(state, theme) {
      state.themeType = theme
    }
  }
}
```

Modify the theme type in vuex when switching themes

```
const handleHeaderTheme = (item) => {
  store.commit('theme/setThemeType', item.type)
}
```

Monitor changes in theme types: theme-light, theme-dark, theme-system, and dynamically set the class attribute value for the html tag. He just adds the css prefix to the corresponding theme to the html element when switching. To achieve the effect of switching themes

```
<html lang="en" class="dark">
    <!-- To add a dark mode CSS style, simply prepend it with the "dark" prefix. -->
    <div class="bg-zinc-300 dark:bg-zinc-900" ></div>
</html>
```

After the value of the class attribute of html changes, it will match the class of the corresponding theme, thereby displaying the color of the corresponding theme.

Set two sets of class names for labels: one for white and one for dark.

```
<div class="bg-zinc-300 dark:bg-zinc-900" ></div>
```

To follow the theme changes of the system, you need to use `Window.matchMedia()`. This method receives a mediaQueryString (a string parsed by the media query). We can pass the prefers-color-scheme to this string, that is, `window.matchMedia('(prefers-color-scheme: dark)')` The method returns a `MediaQueryList` object.

- This object has a `change` event that can listen for system theme changes.

- The event object `matches` attribute can determine the theme. (true: dark theme, false: light theme).

Theme modification tool function

```
import { watch } from 'vue'
import store from '../store'
import { THEME_DARK, THEME_LIGHT, THEME_SYSTEM } from '../constants'

/**
 * Listen for system theme changes
 */
let matchMedia = ''
function changeSystemTheme() {
  // Only need to be initialized once
  if (matchMedia) return
  matchMedia = window.matchMedia('(prefers-color-scheme: dark)')

  // This also listens for theme switching, and then calls to modify the HTML class
  matchMedia.addEventListener('change', (event) => {
    changeTheme(THEME_SYSTEM)
  })
}

/**
 * Theme matching function
 * @param val {*} Theme tag
 */
const changeTheme = (val) => {
  let htmlClass = ''
  if (val === THEME_LIGHT) {
    // Light theme
    htmlClass = THEME_LIGHT
  } else if (val === THEME_DARK) {
    // Dark theme
    htmlClass = THEME_DARK
  } else {
    // Follow the system
    changeSystemTheme()
    // True is dark mode, false is light theme
    htmlClass = matchMedia.matches ? THEME_DARK : THEME_LIGHT
  }
  document.querySelector('html').className = htmlClass
}

/**
 * Initialize the theme
 */
export default () => {
  // Listen for theme switching and modify the value of the HTML class
  watch(() => store.getters.themeType, changeTheme, {
    immediate: true
  })
}
```

## Implement waterfall flow layout

The construction of the entire waterfall flow component generally needs to be divided into several parts

1. Pass key data through props

  - data: data source

  - nodeKey: unique identifier

  - column: number of columns rendered

  - columnSpacing: column spacing

  - rowSpacing: row spacing

  - picturePreReading: Whether picture pre-rendering is required

2. Waterfall rendering mechanism: layout is completed through absolute and relative. The layout logic is: each item should be arranged horizontally, and the items in the second row are sequentially connected to the current shortest column.

3. Pass the key data involved in each item to the item view through the scope slot.

**Calculate the width of each column**

The general calculation method is to get the container width (excluding margin, padding, border),

```
const useContainerWidth = () => {
  const { paddingLeft, paddingRight } = getComputedStyle(
    containerRef.value,
    null
  )
  // Container left margin
  containerLeft.value = parseFloat(paddingLeft)
  // Container width
  containerWidth.value =
    containerRef.value.offsetWidth -
    parseFloat(paddingLeft) -
    parseFloat(paddingRight)
}
```

And get the total spacing of each item element in the container.

```
// Total column spacing size (column - 1) * columnSpacing
const columnSpacingTotal = computed(() => {
  return (props.column - 1) * props.columnSpacing
})
```

Then subtract the total spacing from the current container, divided by the number of columns.

```
const useColumnWidth = () => {
  // Get container width
  useContainerWidth()
  // Get column width
  columnWidth.value =
    (containerWidth.value - columnSpacingTotal.value) / props.column
}
```

**Get the height of each element**

Whether the height of the image is defined. If the height is defined, the height of each item can be calculated directly.

```
const useItemHeight = () => {
  // Initialize item height list
  itemsHeight = []
  // Get item element
  const itemElements = [...document.getElementsByClassName('hm-waterfall-item')]
  // Get item height
  itemElements.forEach((itemEl) => {
    itemsHeight.push(itemEl.offsetHeight)
  })
  // Render position
  useItemLocation()
}
```

If the height is not defined, we need to calculate the height after the image is loaded.

- Get item element

- Get the image path in the itm element

```
/**
 * Get all img elements in item
 */

export function getImgElements(itemElements) {
  const imgElements = []
  itemElements.forEach((el) => {
    imgElements.push(...el.getElementsByTagName('img'))
  })
  return imgElements
}

/**
 * Get all image paths
 */

export function getAllImgSrc(imgElements) {
  const allImgSrc = []
  imgElements.forEach((item) => {
    allImgSrc.push(item.getAttribute('src'))
  })
  return allImgSrc
}
```

- Use the load event of the image object to determine whether the image is loaded, and then calculate the height.

```
export function allImgComplete(allImgSrc) {
  // Store all promise objects for image loading
  const promises = []
  // Loop through allImgSrc
  allImgSrc.forEach((imgSrc, index) => {
    promises.push(
      new Promise((resolve) => {
        const imgObj = new Image()
        imgObj.src = imgSrc
        imgObj.onload = () => {
          resolve({
            imgSrc,
            index
          })
        }
      })
    )
  })
  return Promise.all(promises)
}
```

```
const waitImgComplete = () => {
  // Initialize item height list
  itemsHeight = []
  // Get item elements
  const itemElements = [...document.getElementsByClassName('hm-waterfall-item')]
  // Get all img elements in item
  const imgElements = getImgElements(itemElements)
  // Get all image paths
  const allImgSrc = getAllImgSrc(imgElements)
  // Preload images and then calculate height
  allImgComplete(allImgSrc).then(() => {
    itemElements.forEach((itemEl) => {
      itemsHeight.push(itemEl.offsetHeight)
    })
  })
  // Render location
  useItemLocation()
}
```

**Calculate the offset of each element**

They are all calculated based on obtaining the minimum height of the column.

You need to initialize the height of each column to 0, use this object as a container, the key is the column subscript, and the value is the column height.

```
// The total height of the container
const containerHeight = ref(0)
// Store each column's height. key: column index, val: column height
const columnHeightObj = ref({})
/**
 * Create an object to store the height of each column. Initialize all as 0
 */
const useColumnHeightObj = () => {
  columnHeightObj.value = {}
  for (let i = 0; i < props.column; i++) {
    columnHeightObj.value[i] = 0
  }
}
```

When getting the left offset, we need to get the minimum height column.

```
/**
 * Get the minimum height
 */
export function getMinHeight(columnHeightObj) {
  const columnHeightValue = Object.values(columnHeightObj)
  return Math.min(...columnHeightValue)
}

/**
 * Get the column with the minimum height
 */
export function getMinHeightColumn(columnHeightObj) {
  // Get the minimum height
  const minHeight = getMinHeight(columnHeightObj)
  const columns = Object.keys(columnHeightObj)
  const minHeightColumn = columns.find((col) => {
    return columnHeightObj[col] === minHeight
  })
  return minHeightColumn
}
```

After getting the minimum height column, just multiply it by the column width and add the spacing.

```
/**
 * Calculate the left offset of the current element
 */
const getItemLeft = () => {
  // Get the column with the minimum height
  const column = getMinHeightColumn(columnHeightObj.value)
  // Calculate left
  return (
    (columnWidth.value + props.columnSpacing) * column + containerLeft.value
  )
}
```

To calculate the top offset, we can directly get the minimum height column height.

```
/**
 * Calculate the top offset of the current element
 */
const getItemTop = () => {
  // Get the column's minimum height
  const minHeight = getMinHeight(columnHeightObj.value)
  return minHeight
}
```

It should be noted that when we complete each element offset assignment, we need to recalculate the height of the minimum height column.

```
/**
 * Recalculate the height of the column with the minimum height
 */
const increasingHeight = (index) => {
  // Get the column with the minimum height
  const column = getMinHeightColumn(columnHeightObj.value)
  // Recalculate the height for this column
  columnHeightObj.value[column] = 
    columnHeightObj.value[column] + itemsHeight[index] + props.rowSpacing
}
```

Finally, assign the maximum height column height to the container height.

```
// Render position
const useItemLocation = () => {
  props.data.forEach((item, index) => {
    // Avoid repeated calculation
    if (item._style) return

    // Obtain the minimum height and calculate left, top in _style
    item._style = {}
    item._style.left = getItemLeft()
    item._style.top = getItemTop()
    // Every time the offset is set, the height of the shortest column needs to be updated.
    increasingHeight(index)
  })

  // Once all items have their offsets set, set the container height to the height of the tallest column
  containerHeight.value = getMaxHeight(columnHeightObj.value)
}
```
