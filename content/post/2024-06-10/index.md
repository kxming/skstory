---
title: 'Building an Enterprise-level development scaffold with Vite + Vue3 + TS + Pinia'
date: 2024-06-10T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*EtmBfS6zz62l-m9t2bixkg.png"
categories: ["Code","Javascript"]
tags: ["Code","Javascript"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

## Preparation for Setup

- **Vscode**: A must-have coding tool for frontend developers

- **Chrome**: A browser that is very friendly to developers

- **Nodejs&npm**: Configure the local development environment, installing Node will also install npm

- **Vue.js devtools**: Browser debugging plugin

- **Vue Language Features (Volar)**: Essential Vscode plugin for developing with vue3, offering syntax highlighting tips, very useful

- **Vue 3 Snippets**: Quick input shortcuts for vue3

---

## 1. Initializing the Project

Follow the prompts for initialization:

### 1. Use the `vite-cli` command

```
# pnpm
pnpm create vite

# npm
npm init vite@latest

# yarn
yarn create vite
```

### 2. Enter the project name

```
? Project name:  vite-vue3-ts-pinia
```

### 3. Choose a Framework (Vue)

```
? Select a framework: » - Use arrow-keys. Return to submit.
    vanilla // Vanilla JS
❯   vue // Defaults to Vue3
    react // React
    preact // Lightweight React framework
    lit // Lightweight web components
    svelte // Svelte framework
```

### 4. Use TypeScript

```
? Select a variant: › - Use arrow-keys. Return to submit.
     vue
 ❯   vue-ts
```

### 5. Start the project

```
cd vite-vue3-ts-pinia && pnpm install && pnpm run dev
```

**Quick Initialization (Recommended):**

```
# pnpm
pnpm create vite project-name -- --template vue-ts

# yarn
yarn create vite project-name --template vue-ts
```

## 2. Enforce Code Style

### 1. Eslint Support

```
# eslint installation
yarn add eslint --dev

# eslint plugin installation
yarn add eslint-plugin-vue --dev

# Since ESLint by default uses Espree for syntax parsing, it cannot recognize some syntax of TypeScript. Therefore, we need to install @typescript-eslint/parser to replace the default parser.
yarn add @typescript-eslint/parser --save-dev

# Install the corresponding plugin @typescript-eslint/eslint-plugin, which serves as a supplement to the default rules of eslint, providing some additional rules applicable to TypeScript syntax.
yarn add @typescript-eslint/eslint-plugin --dev

# typescript parser
yarn add @typescript-eslint/parser --dev
```

### 2. Create a new `.eslintrc.js` in the project

Configure eslint verification rules:

```
module.exports = {
  root: true,
  env: {
    browser: true,
    node: true,
    es2021: true,
  },
  parser: 'vue-eslint-parser',
  extends: [
    'eslint:recommended',
    'plugin:vue/vue3-recommended',
    'plugin:@typescript-eslint/recommended',
    'plugin:prettier/recommended',
  ],
  parserOptions: {
    ecmaVersion: 12,
    parser: '@typescript-eslint/parser',
    sourceType: 'module',
    ecmaFeatures: {
      jsx: true,
    },
  },
  // eslint-plugin-vue @typescript-eslint/eslint-plugin eslint-plugin-prettier的缩写
  plugins: ['vue', '@typescript-eslint', 'prettier'],
  rules: {
      ...
  },
  globals: {
    defineProps: 'readonly',
    defineEmits: 'readonly',
    defineExpose: 'readonly',
    withDefaults: 'readonly',
  },
}
```

### 3. Create a new `.eslintignore` in the project

```
# eslint ignore list (add according to project needs)
node_modules
dist
```

### 4. prettier support

```
# Install prettier
yarn add prettier --dev
```

### 5. Resolve conflicts between eslint and prettier

Solve the conflict between style specifications in ESLint and prettier, prioritize the style specifications of prettier, making the style specifications in ESLint automatically invalid

- **eslint-plugin-prettier**: ESLint rules based on Prettier code style, meaning ESLint uses Prettier rules to format code.

- **eslint-config-prettier**: Disables all formatting-related ESLint rules to solve conflicts between Prettier and ESLint rules.

```
# Install plugin eslint-config-prettier eslint-plugin-prettier
yarn add eslint-config-prettier eslint-plugin-prettier --dev
```

Modify the .eslintrc.js configuration

```
module.exports = {
    ...

    extends: [
        ...
        // shorthand for eslint-config-prettier
        'prettier',
        'plugin:prettier/recommended'
    ],

    ...
};
```

### 6. Create a new `.prettier.js` in the project

Configure prettier formatting rules:

```
module.exports = {
  tabWidth: 2,
  jsxSingleQuote: true,
  jsxBracketSameLine: true,
  printWidth: 100,
  singleQuote: true,
  semi: false,
  overrides: [
    {
      files: '*.json',
      options: {
        printWidth: 200,
      },
    },
  ],
  arrowParens: 'always',
}
```

### 7. Create a new `.prettierignore` in the project

```
# Ignore formatting for files (add according to project needs)
node_modules
dist
```

### 8. package.json configuration:

```
{
  "script": {
    "lint": "eslint src --fix --ext .ts,.tsx,.vue,.js,.jsx",
    "prettier": "prettier --write ."
  }
}
```

After the above configuration is complete, you can run the following commands to test the code check and formatting effect:

```
# eslint check
yarn lint

# prettier automatic formatting
yarn prettier
```

### 9. Configuring `husky` + `lint-staged`

Use `husky` + `lint-staged` to assist in team coding standards. It will install and configure `husky` and `lint-staged` based on the code quality tools in the package.json dependencies. Therefore, please ensure all code quality tools such as Prettier and ESLint are installed and configured beforehand.

`husky` is a tool that adds hooks to the `git` client. After installation, it automatically adds the corresponding hooks in the `.git/` directory of the repository; for example, the `pre-commit` hook is triggered when you execute `git commit`.

Thus, we can implement operations such as lint checks, unit testing, and code beautification in the `pre-commit`. Of course, the commands executed during the `pre-commit` stage should ensure they are not too slow, as waiting a long time for each commit is not a pleasant experience.

`lint-staged` is a tool that only filters out files in the Git staging area (files that have been `git add`); this is very practical because if we perform a check on the entire project’s code, it may take a long time, especially if it’s an old project, to check and modify the code according to coding standards for previous code, which could be troublesome and potentially lead to significant changes in the project.

Therefore, `lint-staged` is a very useful tool for team and open-source projects, as it provides a standard and constraint for the code that an individual intends to submit.

#### Install

```
pnpm i husky lint-staged -D
```

#### `package.json` configuration

```
"husky": {
    "hooks": {
      "pre-commit": "lint-staged"
    }
  },
  "lint-staged": {
    "*.{js,jsx,vue,ts,tsx}": [
      "yarn lint",
      "prettier --write",
      "git add"
    ]
  }
```

### 10. Configuring file reference aliases (alias)

Directly modify the configuration in the `vite.config.ts` file:

```
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import path from 'path'

// https://vitejs.dev/config/
export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': path.resolve(__dirname, 'src'),
    },
  },
})
```

Modify `tsconfig.json`

```
{
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    "moduleResolution": "node",
    "strict": true,
    "jsx": "preserve",
    "sourceMap": true,
    "resolveJsonModule": true,
    "esModuleInterop": true,
    "lib": ["esnext", "dom"],
    "baseUrl": ".",
    "paths": {
      "@/*":["src/*"]
    }
  },
  "include": ["src/**/*.ts", "src/**/*.d.ts", "src/**/*.tsx", "src/**/*.vue"]
}
```

## 3. Configuration of Environment Variables

Vite offers two modes: a development mode with a development server (development) and a production mode (production).

Create in the project root directory: `.env.development`:

```
NODE_ENV=development

VITE_APP_WEB_URL= 'YOUR WEB URL'
```

Create in the project root directory: `.env.production`:

```
NODE_ENV=production

VITE_APP_WEB_URL= 'YOUR WEB URL'
```

Usage in components:

```
console.log(import.meta.env.VITE_APP_WEB_URL)
```

Configuration in `package.json`:

Differentiate between development and production environments for building

```
"build:dev": "vite build --mode development",
"build:pro": "vite build --mode production",
```

## 4. Integrating Pinia

`Pinia`, is a lightweight state management library recommended by the Vue official team as a replacement for `Vuex`.

`Pinia` has the following features:

- Complete TypeScript support;

- Sufficiently lightweight, with a compressed size of only 1.6kb;

- Removes mutations, leaving only state, getters, and actions (this is my favorite feature);

- Actions support both synchronous and asynchronous operations;

- No module nesting, only the concept of store, which can be freely used between stores for better code splitting;

- No need to manually add stores, stores are automatically added once created;

### Installation

```
yarn add pinia
```

### Usage

1. Create a directory `src/store` and under it create `index.ts`, exporting store

```
import { createPinia } from 'pinia'

const store = createPinia()

export default store
```

2. Incorporate and use it in `main.ts`

```
import { createApp } from 'vue'
import App from './App.vue'
import store from './store'

// Creating the vue instance
const app = createApp(App)

// Mounting pinia
app.use(store)

// Mounting the instance
app.mount('#app');
```

3. Defining State: Create a `user.ts` under `src/store`

```
import { defineStore } from 'pinia'

export const useUserStore = defineStore({
  id: 'user', // id is required and needs to be unique
  state: () => {
    return {
      name: 'Zhang San'
    }
  },
  actions: {
    updateName(name) {
      this.name = name
    }
  }
})
```

4. Getting State: Use it in `src/components/usePinia.vue`

```
<template>
  <div>{{ userStore.name }}</div>
</template>

<script lang="ts" setup>
import { useUserStore } from '@/store/user'

const userStore = useUserStore()
</script>
```

5. Modifying State:

```
// 1. Directly modify the state (not recommended)
userStore.name = 'Li Si'

// 2. Modify through actions
<script lang="ts" setup>
import { useUserStore } from '@/store/user'

const userStore = useUserStore()
userStore.updateName('Li Si')
</script>
```

## 5. Integrating Vue-router

### Installation

```
yarn add vue-router
```

### Usage

Create directory `src/router` and create `index.ts` under it, export router

```
import { createRouter, createWebHistory, RouteRecordRaw } from 'vue-router';

const routes: Array<RouteRecordRaw> = [
  {
    path: '/login',
    name: 'Login',
    meta: {
        title: 'Login',
        keepAlive: true,
        requireAuth: false
    },
    component: () => import('@/pages/login.vue')
  },
  {
      path: '/',
      name: 'Index',
      meta: {
          title: 'Home',
          keepAlive: true,
          requireAuth: true
      },
      component: () => import('@/pages/index.vue')
  }
]

const router = createRouter({
  history: createWebHistory(),
  routes
});
export default router;
```

Incorporate and use it in `main.ts`

```
import { createApp } from 'vue'
import App from './App.vue'
import store from './store'
import router from '@/router';

// Creating the vue instance
const app = createApp(App);

app.use(router);

// Mounting the instance
app.mount('#app');
```

Modify `App.vue`

```
 <template>
   <RouterView/>
 </template>
```

## 6. Integrating VueUse

VueUse is a collection of utility functions based on the Composition API.

### Installation

```
yarn add @vueuse/core
```

### Usage

Create a new `src/page/vueUse.vue` page for a simple demo

```
<template>
  <h1> Testing vueUse mouse coordinates </h1>
  <h3>Mouse: {{x}} x {{y}}</h3>
</template>

<script lang="ts">
    import { defineComponent } from 'vue';
    import { useMouse } from '@vueuse/core'

    export default defineComponent({
        name: 'VueUse',
        setup() {
          const { x, y } = useMouse()

          return {
            x, y
          }
        }
    });
</script>
```

`useMouse` is just one of the most basic libraries in `vueuse`, and there are many others, there is always one that suits you.

For more functions official documentation: [link](https://link.juejin.cn/?target=https%3A%2F%2Fvueuse.org%2F)

## 7. Integration of CSS

### Option 1: Native css variable new feature:

Natively supported, no third-party plugins required, for detailed usage documentation see

Create a file `src/styles/index.css`

```
:root {
  --main-bg-color: pink;
}
​
body {
  background-color: var(--main-bg-color);
}
```

**Note**: You can also add PostCSS configuration, (any format supported by `postcss-load-config`, such as `postcss.config.js`), it will automatically apply to all imported CSS.

### Option 2: scss or less:

#### Installation

```
# .scss and .sass
pnpm add -D sass

# .less
pnpm add -D less
```

### Option 3: tailwindcss

#### Installation

```
yarn add -D tailwindcss postcss autoprefixer
```

#### Configuration

##### 1. Initialize

```
npx tailwindcss init -p
```

It will generate two files, `postcss.config.js` and `tailwind.config.js` in the root directory.

##### 2. Add tailwindcss and autoprefixer to your `postcss.config.js` file

```
export default {
   plugins: {
      tailwindcss: {},
      autoprefixer: {}
   }
};
```

##### 3. Add the paths to all of your template files in your `tailwind.config.js` file.

```
/** @type {import('tailwindcss').Config} */
module.exports = {
  darkMode: "class",
  corePlugins: {
    preflight: false
  },
  content: ["./index.html", "./src/**/*.{vue,js,ts,jsx,tsx}"],
  theme: {
    extend: {
      colors: {}
    }
  }
};
```

##### 4. Configure `vite.config.ts` options

```
export default defineConfig({
  css: {
      postcss: {
        plugins: [
          require('tailwindcss'),
          require('autoprefixer'),
        ],
      },
  },
});
```

Using the configuration above to import `tailwindcss`, you might encounter the error `Dynamic require of ‘tailwindcss’ is not supported`, which is usually because Vite does not support the dynamic require of `CommonJS` when trying to import modules. Vite uses the `ES module` system and expects all dependencies to be in ESM format. Therefore, I used the following method.

```
import postcssImport from "postcss-import"
import tailwindcss from "tailwindcss"
import autoprefixer from "autoprefixer"

export default defineConfig({
  css: {
    postcss: {
      plugins: [
        // First run the postcss-import plugin to ensure @import statements are correctly resolved
        postcssImport(),
        // Then apply TailwindCSS
        tailwindcss("./tailwind.config.js"),
        // Finally, add autoprefixer for compatibility processing
        autoprefixer()
      ]
    }
  }
})
```

If the `postcss-import` module cannot be found, execute the following command

```
yarn add -D @types/postcss-import
```

##### 5. Create a new `tailwind.css` file in `src/style`.

```
@tailwind base;
@tailwind components;
@tailwind utilities;
```

##### 6. Import in `main.ts`

**Note**: It’s essential to import tailwind.css in main.ts to prevent vite from requesting the whole css file from `src/style/index.scss` on every HMR, which causes slow hot updates.

#### Usage

```
<template>
  <div class="h-[200px] w-[200px] bg-[#ccc]">
    <span>123 </span>
    <span class="ml-[100px] text-[#999]">456 </span>
  </div>
</template>
```

#### VsCode Plugin

Install `Tailwind CSS IntelliSense`

## 8. Integrating axios

`axios` is a promise-based HTTP library that can be used in both browser and node.js.

### Installation

```
yarn add axios
```

### Usage

Create a new `src/utils/axios.ts`

```
import axios, { AxiosResponse, AxiosRequestConfig } from 'axios';

const service = axios.create();

// Request interceptors
service.interceptors.request.use(
    (config: AxiosRequestConfig) => {
        // do something
        return config;
    },
    (error: any) => {
        return Promise.reject(error);
    }
);

// Response interceptors
service.interceptors.response.use(
    async (response: AxiosResponse) => {
        // do something
    },
    (error: any) => {
        // do something
        return Promise.reject(error);
    }
);

export default service;
```

You can use it on the page.

```
<script lang="ts">
    import request from '@/utils/axios';
    const requestRes = async () => {
        let result = await request({
                    url: '/api/xxx',
                    method: 'get'
                  });
    }

</script>
```

**Encapsulate all APIs for request parameters and response data (optional)**

Create a new `src/api/index.ts`

```
import * as login from './module/login';
import * as index from './module/index';

export default Object.assign({}, login, index);
```

Create new `src/api/module/login.ts` and `src/api/module/index.ts`

```
import request from '@/utils/axios';

/**
 * Login
 */
 
interface IResponseType<P = {}> {
    code?: number;
    status: number;
    msg: string;
    data: P;
}
interface ILogin {
    token: string;
    expires: number;
}
export const login = (username: string, password: string) => {
    return request<IResponseType<ILogin>>({
        url: '/api/auth/login',
        method: 'post',
        data: {
            username,
            password
        }
    });
};
```

Since TypeScript is used, you need to add a new `src/types/shims-axios.d.ts`

```
import { AxiosRequestConfig } from 'axios';
/**
 * Custom extension of the axios module
 * @author Maybe
 */
declare module 'axios' {
    export interface AxiosInstance {
        <T = any>(config: AxiosRequestConfig): Promise<T>;
        request<T = any>(config: AxiosRequestConfig): Promise<T>;
        get<T = any>(url: string, config?: AxiosRequestConfig): Promise<T>;
        delete<T = any>(url: string, config?: AxiosRequestConfig): Promise<T>;
        head<T = any>(url: string, config?: AxiosRequestConfig): Promise<T>;
        post<T = any>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T>;
        put<T = any>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T>;
        patch<T = any>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T>;
    }
}
```

