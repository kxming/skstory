---
title: 'Stop flooding the screen with import statements！！！'
date: 2024-05-21T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*0zTKOnJL-nspQnB9oGhIjA.png"
categories: ["Code","Javascript"]
tags: ["Code","Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "What would it be like to open a file and see nothing but a screen full of import statements?"
---

What would it be like to open a file and see nothing but a screen full of import statements?

The dense import statements are not just a visual assault, but also a test of the code’s organizational structure.

How did we end up with a situation where imports “take over the screen,” and how can we manage these import statements elegantly?

## Using Re-export

Module re-export is a common technique. For instance, the front-end component library [arco-design](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Farco-design%2Farco-design%2Fblob%2Fmain%2Fcomponents%2Findex.tsx) exposes all components in the components/index.tsx file, allowing the use of N components with a single import when utilizing the library.

### 1. Direct Re-export

Directly re-export specific members from another module.

```
export { foo, bar } from './moduleA';
```

### 2. Rename and Re-export (Including Default Exports)

Import members from another module, potentially rename them and then export again.

Default exports can also be renamed and re-exported

```
// Exported through export
export { foo as newFoo, bar as newBar } from './moduleA';
// Exported through export default
export { default as ModuleDDefault } from './moduleD';
```

### 3. Re-export the Entire Module (Excluding Default Exports)

Re-export all exported members of another module as a single object. (Note: The entire export will not include export default)

```
export * from './moduleA';
```

### 4. Combine Importing and Re-exporting

First, import members from a module, use them, and then re-export them.

```
import { foo, bar } from './moduleA';
export { foo, bar };
```

Through these forms, we can flexibly organize and manage code modules. Each form has its applicable scenarios, and choosing the appropriate method can help us build a clearer and more efficient code structure.

## Using require.context

`require.context` is a very useful feature allowing us to dynamically import a set of modules without the need to explicitly import each one by one.

A single piece of code enables automatic collection and re-exporting simply by adding files or components.

It’s extremely useful in fixed scenarios such as project routing, state management, etc., (achieving efficiency, and avoiding the need to modify N files when adding a single configuration as much as possible)

Especially when configuring routes, it generates a large number of imports (as many pages as there are, that’s how many imports you must include)

```
// Without using require.context
import A from '@/pages/A'
import B from '@/pages/B'
...
// routes/index.ts file handles everything uniformly
// Create a context to import all .ts files under the routes directory
const routesContext = require.context('./routes', false, /.ts$/);
const routes = [];
// Iterate through each module in the context
routesContext.keys().forEach(modulePath => {
  // Get the module's export
  const route = routesContext(modulePath);
  // Get the component name [if needed], for example, extract "Header" from "./Header.ts"
  // const routeName = modulePath.replace(/^./(.*).\w+$/, '$1');
  // Store the component in the components object
  routes.push(route.default || route)
});

export default routes;
```

In large projects with multiple routes, using require.context for routing importation can be very advantageous.

## Using Dynamic Import

Dynamic import can achieve functionality similar to require.context, dynamically collecting modules. Dynamic `import()` is part of the ES2020 specification and is widely supported by modern browsers. It allows developers to load modules on-demand during code execution. Unlike static imports (for example, using the import … from … syntax), dynamic import can be triggered at any point in time, providing great flexibility and efficiency for code splitting, on-demand loading, lazy loading, and conditional loading.

Usage:

```
// Assuming we dynamically import a module
import('path/module').then((module) => {
  // Use the module
}).catch(console.error);
// If loading the module fails, it can be handled in the catch.
```

The `import()` function returns a Promise, allowing the imported module to be used within the Promise’s then method. Unlike static imports, `import()` enables dynamic loading of modules, meaning modules can be loaded on-demand at runtime rather than all at once during the code loading phase.

Each time the `import()` function is invoked within a file, a new Promise is created. This Promise represents the process of loading and parsing the corresponding module, implying that each file gets a distinct module instance. However, in reality, the entire application only has a single instance of the module.js module.

This occurs because when the first file calls the `import()` function, it begins loading and parsing the module.js module. When the second file calls `import()`, since the module has already been loaded and parsed, it won’t load and parse again but rather returns the already loaded and parsed module instance. This means that if `import()` is called multiple times within an application to import the same module, only the first call will actually perform the loading and parsing operation; subsequent calls will simply return the cached module instance.

It’s important to note that since `import()` is asynchronous, the variables or functions exported by the module cannot be used until the module has finished loading. Therefore, when importing modules using `import()`, mechanisms such as Promises or async/await should be used to handle asynchronous operations.

**Note**:

### 1. Relative Path vs Absolute Path vs Module Resolution

**Relative Path**: Paths starting with `.` or `..` are considered relative paths. `./` indicates the current directory, while `../` indicates the parent directory. When using relative paths, the imported module is resolved in relation to the location of the current file.

**Absolute Path**: Paths not starting with `.` or `..` are considered absolute paths. In Node.js, an absolute path could be a path that starts from the project’s root directory, or a path to a module inside the node_modules directory.

**Module Resolution**

When the path starts with `./` or `../`, the JavaScript runtime (or bundling tools, such as Webpack, Vite, etc.) considers it a relative path relative to the current file and attempts to resolve the module under that path.
If the path does not start with `./`, `../`, or `/`, it is usually interpreted as a package name or built-in module, and the runtime will try to find the module in the node_modules directory or the corresponding built-in module library.

```
// Cannot be resolved (treated as module resolution)
const name = `${filename.replace(directoryPath + '/', './')}`
import(`${name}`)
// Can be resolved (treated as a relative path resolution)
const name = `${filename.replace(directoryPath + '/', '')}`
import(`./${name}`)
```

### 2. Characteristics of Dynamic import()

When using dynamic `import()` to import a module, if it does not start with `./` or `../`, the JavaScript runtime might attempt to treat it as a package or built-in module, rather than a relative path relative to the current file. This is usually not the desired behavior, especially when you are trying to dynamically import a module located in the same directory as the current file.

## Using ProvidePlugin

`webpack.ProvidePlugin` is a powerful tool but shouldn’t be overused.

Variables/functions/libraries or tools used in the project can be utilized anywhere once configured.

Trust me — after seeing this example, if you haven’t used it, you’ll definitely be eager to try it out

```
const webpack = require('webpack');
module.exports = {
  // Other configurations...
  plugins: [
    new webpack.ProvidePlugin({
      React: 'react',
      _: 'lodash',
      dayjs: 'dayjs',
      // Suppose the self-defined utils.js is under the src directory
      Utils: path.resolve(__dirname, 'src/utils.js')
    })
  ]
  // Other configurations...
};
```

Now you can use dayjs, lodash, Utils, etc., anywhere without needing to import them.

In summary:

- `webpack.ProvidePlugin` is a powerful tool that can help reduce redundant import statements, making the code cleaner and neater. However, it doesn’t reduce the build size since these libraries will still be included in your final bundle. Properly using this plugin can improve development efficiency, but it should be used cautiously to avoid hiding dependencies that make the code hard to understand and maintain.

- For modules or components that need to be loaded on-demand, consider using the dynamic `import()` syntax, allowing for more effective control of the code’s loading time and reducing the bundle size.

- Use ProvidePlugin with caution, only configure global variables for modules that truly need to be used in multiple places, to avoid unnecessary code bundling.

Additionally, if it’s a Vite project, you can use `vite-plugin-inject` to replace the functionality of `ProvidePlugin`.

```
// Configuration
import inject from 'vite-plugin-inject'; // Currently unavailable, will update with an alternative solution
...
plugins: [
  inject({
    // The key is the global variable you want to provide, the value is the module you're providing
    dayjs: 'dayjs', // For example, this will provide 'dayjs' globally, accessible via dayjs
    // You can continue adding other modules you want to provide globally
  }),
]
...
```

If you’re using TS, remember to configure the types:

```
// globals.d.ts file for handling global types
import dayjs from 'dayjs';
declare global {
  const dayjs: typeof dayjs;
}
// Also configure the tsconfig.json file
{ 
  "compilerOptions": { 
    // Compilation options... 
  }, 
  "include": [ "src/**/*", "globals.d.ts" // Ensure TypeScript includes this file ] 
}
```

## Using TypeScript to Import Types

In a TS project, imports are inevitable, including those of TypeScript. However, if configured properly, the need for imports can be significantly reduced.

Here, I’ll introduce the method I use most frequently in my projects: TypeScript namespaces. With it, types can be modularized, and even more incredibly, types can be used directly without import.

Similarly, it eliminates the need for import imports just like ProvidePlugin does.

Usage example:

```
// accout.ts
declare namespace IAccount {
  type IList<T = IItem> = {
    count: number
    list: T[]
  }
  interface IUser {
    id: number;
    name: string;
    avatar: string;
  }
}
// Directly use in any file, no need to import
const [list, setList] = useState<IAccount.IList|undefined>();
const [user, setUser] = useState<IAccount.IUser|undefined>();
```

Note: eslint might need to be configured to enable the use of namespaces.

## Other

### 1. Setting webpack and ts aliases

It can both shorten the import path and make it more semantically meaningful.

```
resolve: {
  alias: {
    "@src": path.resolve(__dirname, 'src/'),
    "@components": path.resolve(__dirname, 'src/components/'),
    "@utils": path.resolve(__dirname, 'src/utils/')
  }
}

// Before using alias
import MyComponent from '../../../../components/MyComponent';

// After using alias
import MyComponent from '@components/MyComponent';
```

### 2. Setting the format of prettier.printWidth

A too small value might cause frequent line breaks, making it difficult to read. A value of 120 seems more suitable (based on the actual usage of the team).

```
{
  "printWidth": 120,
  ...
}
```

### 3. Dynamically loading components globally based on conditions

Import global components in the entry file, using require.ensure or import to dynamically load components based on conditions, which facilitates maintenance, reduces references, and decreases performance overhead.

```
// Asynchronously load global dialogs to reduce performance overhead
Vue.component('IMessage', function (resolve) {
  // Specify conditions for global loading, no need to reference in specific pages
  if (/^\/pagea|pageb/.test(location.pathname)) {
    require.ensure(['./components/message/index.vue'], function() {
      resolve(require('./components/message/index.vue'));
    });
  }
});
```

### 4. Using babel-plugin-import

`babel-plugin-import` doesn’t directly reduce the number of imports, but optimizes import statements to reduce the bundle size and improve the project’s loading performance. This is a very valuable optimization for projects that use large third-party libraries.

Taking arco-design as an example:

```
// .babelrc configuration
{
  "plugins": [
    ["import", {
      "libraryName": "@arco-design/web-react",
      "libraryDirectory": "es", // or "lib", depending on the module system used
      "style": true // Load CSS
    }, "@arco-design/web-react"]
  ]
}
// This configuration tells babel-plugin-import to automatically transform import statements like import { Button } from '@arco-design/web-react'; into an on-demand import form, and to load the corresponding CSS file.
```

```
// Use in business
import { Button } from '@arco-design/web-react';
// Will be compiled by babel into
import Button from '@arco-design/web-react/es/button';
// If style is set to true
import '@arco-design/web-react/es/button/style/css.js';
```
