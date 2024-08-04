---
title: 'Using Tailwind CSS in Vue3'
date: 2024-08-01T16:02:05+08:00
draft: false
author: ""
image: "https://cdn-images-1.medium.com/max/1600/1*3LVaHw2SN2wh5JhwDZHHyA.png"
categories: ["Story","Fiction", "Life"]
tags: ["Story","Fiction", "Life"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

When it comes to frontend development frameworks, [Tailwind CSS](https://medium.com/r/?url=https%3A%2F%2Fgithub.com%2Ftailwindlabs%2Ftailwindcss) is a highly prominent choice. It is a powerful and flexible CSS framework that provides a plethora of utility classes to help developers rapidly build modern user interfaces. Tailwind CSS is a popular modern CSS framework that offers a set of atomic classes to construct web interfaces. Unlike traditional CSS frameworks like Bootstrap or Foundation, Tailwind CSS emphasizes a utility-first approach to styling, allowing developers to flexibly combine and customize styles without writing custom CSS.

Here are some key features and concepts of Tailwind CSS:

- **Atomic Classes**: The core concept of Tailwind CSS is atomic classes, which provide a large number of class names, each corresponding to a specific style property. By combining these atomic classes, developers can quickly create the desired styles, such as `bg-red-500` for setting the background color to red, and `text-xl` for setting the text size to extra-large.

- **Utility Classes**: In addition to common style properties, Tailwind CSS offers a wealth of utility classes for layout, spacing, borders, and more. These utility classes help developers quickly achieve responsive design and layouts.

- **Customization**: Tailwind CSS provides extensive configuration options, allowing developers to customize styles according to project requirements, including colors, fonts, spacing, etc. This makes the styles of each project highly customizable.

- **Responsive Design**: Tailwind CSS has built-in utility classes for responsive design, making it easy for developers to write styles that adapt to different screen sizes.

- **Plugin System**: Tailwind CSS features a powerful plugin system that allows developers to write custom plugins to extend the framework's functionality, such as adding new style classes or utility classes.

Overall, Tailwind CSS offers a very flexible way to build web interfaces. It differs significantly from traditional CSS frameworks in its approach, emphasizing atomic style combinations and customization. Many developers find that using Tailwind CSS can significantly increase development efficiency and make styles more maintainable and predictable.

## Installation and Configuration

To start using Tailwind CSS, you first need to install it via npm or yarn. Since Tailwind is a PostCSS plugin, it can also be used with Sass, Less, Stylus, or other preprocessors. Note that you don’t need to use a preprocessor with Tailwind—in any case, you usually write very little CSS in a Tailwind project, so using a preprocessor is not as necessary as it is when writing a lot of custom CSS.

```
npm install -D tailwindcss postcss autoprefixer
# or
yarn add -D tailwindcss postcss autoprefixer
```

After installation, generate the default configuration files and stylesheet by running:

```
npx tailwindcss init -p
```

The generated default configuration files are named `tailwind.config.js` and `postcss.config.js`.

```
/** tailwind.config.js */
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    // Manually add your project directories
    "./index.html",
    "./src/**/*.{vue,js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {},
  },
  plugins: [],
}
```

You can customize properties such as colors, fonts, and spacing in these configuration files. Next, create a CSS file and import Tailwind CSS styles:

```
/* Import Tailwind CSS */
@import 'tailwindcss/base';
@import 'tailwindcss/components';
@import 'tailwindcss/utilities';
```

## Import tailwind.css in main.js

After configuring, import `tailwind.css` by modifying the `src/main.js` file:

```
import './assets/main.css'
import './assets/tailwind.css'

import { createApp } from 'vue'
import { createPinia } from 'pinia'

import App from './App.vue'
import router from './router'

const app = createApp(App)

app.use(createPinia())
app.use(router)

app.mount('#app')
```

## Using Utility Classes

Tailwind CSS provides a rich set of utility classes covering various style properties. Let's demonstrate how to use these utility classes with a simple example. Suppose we want to create a button with a blue background and centered text; we can write:

```
<button class="bg-blue-500 text-white font-bold py-2 px-4 rounded">
  Click me
</button>
```

In this example, we use the `bg-blue-500` class to set the button's background color to blue, the `text-white` class to set the text color to white, the `font-bold` class to set the text to bold, `py-2` and `px-4` classes to set vertical and horizontal padding, and the `rounded` class to set rounded borders.

## Installing the VSCode Plugin

If you decide to use Tailwind CSS, install the [Tailwind CSS IntelliSense](https://medium.com/r/?url=https%3A%2F%2Fmarketplace.visualstudio.com%2Fitems%3FitemName%3Dbradlc.vscode-tailwindcss) plugin provided by the official team. This plugin offers features like autocompletion and real-time preview of actual styles.

![](https://cdn-images-1.medium.com/max/1600/1*cEY-uQaMRsBh9MXtyRWaLg.png)

## Common Plugins

- [autoprefixer](https://medium.com/r/?url=https%3A%2F%2Fgithub.com%2Fpostcss%2Fautoprefixer): Automatically adds vendor prefixes to CSS properties that need them.

- [postcss-import](https://medium.com/r/?url=https%3A%2F%2Fgithub.com%2Fpostcss%2Fpostcss-import): Allows us to use the @import syntax to import CSS files into other files.

- [postcss-preset-env](https://medium.com/r/?url=https%3A%2F%2Fwww.npmjs.com%2Fpackage%2Fpostcss-preset-env): Transforms modern CSS (like nesting and custom media queries) into CSS that browsers can understand.

- postcss-nested: Provides support for writing nested styles in CSS. It is directly included within the Tailwind CSS package itself. By default, it uses the postcss-nested plugin. If you prefer to use postcss-nesting, you can enable it by adding `'tailwindcss/nesting': 'postcss-nesting'` in your configuration file.

- [cssnano](https://medium.com/r/?url=https%3A%2F%2Fcssnano.github.io%2Fcssnano): For CSS code compression.

```
// postcss.config.js
module.exports = {
  plugins: {
    'postcss-import': {},
    'tailwindcss/nesting': {},
    tailwindcss: {},
    autoprefixer: {},
    ...(process.env.NODE_ENV === 'production' ? { cssnano: {} } : {})
  }
}
```

## Configuration

```
/** @type {import('tailwindcss').Config} */
export default {
  // The content section is where you configure the paths to all of your HTML templates, JS components, and any other files that contain Tailwind class names.
  content: [
  ],
  // The theme section is where you define your color palette, fonts, font sizes, border sizes, breakpoints—anything related to the visual design of your site.
  theme: {
    extend: {},
  },
  // The plugins section allows you to register plugins with Tailwind that can be used to generate extra utilities, components, base styles, or custom variants.
  plugins: [],
  // The presets section allows you to specify your own custom base configuration instead of using Tailwind’s default base configuration.
  presets: [
    require('@acmecorp/base-tailwind-config')
  ],
  // The prefix option allows you to add a custom prefix to all of Tailwind’s generated utility classes. This can be really useful when layering Tailwind on top of existing CSS where there might be naming conflicts.
  prefix: 'tw-',
  // The important option lets you control whether or not Tailwind’s utilities should be marked with !important. This can be really useful when using Tailwind with existing CSS that has high specificity selectors.
  important: true,
  // The separator option lets you customize which character should be used to separate modifiers (screen sizes, hover, focus, etc.) from utility names (text-center, items-end, etc.).
  separator: '_',
  // The corePlugins section lets you completely disable classes that Tailwind would normally generate by default if you don’t need them for your project.
  corePlugins: {
    float: false,
    objectFit: false,
    objectPosition: false,
  }
}
```

## Limitation

- The granularity of Tailwind CSS can make the HTML structure overly complex, or require too many class names to achieve a particular effect, making it less readable.

- For developers unfamiliar with Tailwind CSS, the learning curve can be somewhat steep.

[daisyUI](https://medium.com/r/?url=https%3A%2F%2Fdaisyui.com%2F) is a customizable component library for Tailwind CSS. Unlike commonly used libraries like ElementUI or AntDesign, it provides some class names similar to Bootstrap. For ready-to-use components, you need to wrap them yourself. It comes with 47 components and 29 themes, which can be used directly in developed websites. It also supports customization, greatly simplifying the development process.

![](https://cdn-images-1.medium.com/max/1600/1*QIRjHE_yIU_YkNjc1h-Nhg.png)
