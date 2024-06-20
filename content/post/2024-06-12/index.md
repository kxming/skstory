---
title: 'Deep Understanding of Provide/Inject in Vue 3'
date: 2024-06-12T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*lYfmoRGWanDnm0jzf-3rKQ.png"
categories: ["Code","Javascript"]
tags: ["Code","Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "provide and inject are primarily offered for use cases in high-order plugins/component libraries. It’s not recommended for direct use in application code."
---

## Introduction

`provide` and `inject` are primarily offered for use cases in high-order plugins/component libraries. It’s not recommended for direct use in application code.

- **Definition explanation**: These options are used together to allow an ancestor component to inject a dependency into all of its descendants, regardless of the depth of the component hierarchy, and to be effective for the duration of their parent-child relationship.

- **In simpler terms**: If the component hierarchy is too complex, and our descendant components want to access resources from an ancestor component, what should we do? It’s not practical to keep reaching up to the parent level, as this can make the code structure confusing. This is where these options come into play.

  - `provide`: Is an object, or a function that returns an object. It contains things to be given to the descendants, that is, properties and property values.

  - `inject`: Is an array of strings, or an object. The property value can be an object, containing a ‘from’ field and a default value.

In a nutshell: `provide` can offer data and methods to modify it to all descendant components, which use `inject` to access the data.

## Usage

`Parent.vue`:

```
<template>
  <div>
    <son />
  </div>
</template>

<script setup>
import son from "./son.vue";
import { provide } from "vue";
provide("abc", "123");
</script>
```

`Child.vue`:

```
<template>
  <div>
    <grandson />
  </div>
</template>

<script setup>
import grandson from "./DeepChild.vue";
</script>
```

`DeepChild.vue`:

```
<template>
  <div>DeepChild</div>
</template>

<script setup>
import { inject } from "vue";
const abc = inject("abc");
console.log(abc);
</script>
```

**Note:**

> inject() can only be used inside setup() or functional components.

Since adopting the organization method of `Provide/Inject`, the flexibility of the code organization has significantly increased. However, this rise in flexibility is accompanied by a decrease in code fault tolerance. I believe that those who have truly incorporated `Provide/Inject` into their projects must have encountered or are currently experiencing the following situations:

- Often misspelling the injection key, or finding it difficult to name the injection key due to having too many (a common issue among programmers).

- In order to understand what is injected by `inject()`, one has to find the corresponding provide().

- Another scenario is repeatedly `provide()`ing the same value, leading to Injection override.

- When using `inject()`, there may not necessarily be a corresponding `provide()` up the ancestor chain, making it necessary to handle null values or provide default values.

- Using `provide()` within a hook, but components calling this hook cannot `inject()` the provided value from this hook.

…

## What problem does Provide/Inject solve?

Dependency Injection | It is mentioned in Vue.js that the main purpose of these two APIs, Provide and Inject, is to solve the issue of prop drilling (as shown below).

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Yk3pVzaCllDs0pw9yKaBmg.png)

After introducing Provide/Inject, Props can be directly passed to descendant components (as shown below).

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*--koHxOmrxxQ-aIiB5NQPQ.png)

In the root component, the injection value is provided using provide. Here is an example code:

```
import { provide } from 'vue';

provide(/* Injection name */ 'account', /* Value */ { name: 'youth' });
```

In descendant components, the value injected by ancestor components is obtained through inject. Here is an example code:

```
import { inject } from 'vue';

const message = inject('account');
```

## Name Conflict

The issue is how to ensure that `account` isn’t overwritten by other business components? For example, if some business component also supplies information for `account`, as shown below:

```
// Main.vue
...
provide('account', {...})
...

// ParentView.vue
...
provide('account', {...})
...

// DeepChild.vue
...
inject('account')
...
```

The `ParentView` component in the middle layer might be a user list component, which also provides account data. Here, the `account` could be the user selected from the list, whereas the account provided in Main is the current user. In the `DeepChild` component, there might be a need for both the current logged-in user information and the selected user information from the list. However, as it stands, `DeepChild` can only access the selected user information provided by `ParentView`.

Of course, there are many solutions to this business scenario, but let’s assume for now it can only be solved using `provide/inject`.

Certainly, we can simply rename the injection name to `selectAccount` in `ParentView` to solve this problem. But what if there are other components in the middle layer that also have `selectAccount`?

### Solution

Create a file named `injection-key.ts` in the project. I prefer to create this file as `src/constants/injection-key.ts`. This way, the injection names under the project are managed uniformly in this file, and Symbols are used to create injection names, to avoid naming conflicts.

```
export const CurAccountKey = Symbol('account');

export const AuthAccountKey = Symbol('account');
```

Example of usage:

`Main.vue`:

```
import { provide } from 'vue';
import { CurAccountKey } from '@/constants/injectionKeys';

const user = reactive({ id: 1, name: 'youth' });
provide(CurAccountKey, user);
```

`ParentView.vue`:

```
import { provide } from 'vue';
import { AuthAccountKey } from '@/constants/injectionKeys';

const user = reactive({ id: 1, name: 'John Doe' });
provide(AuthAccountKey, user);
```

`DeepChild.vue`:

```
import { inject } from 'vue';
import { AuthAccountKey, CurAccountKey } from '@/constants/injectionKeys';

const curAccount = inject(CurAccountKey);
const authAccount = inject(AuthAccountKey);
```

## Injection Note

But what kind of data will using `inject(CurAccountKey)` bring? This requires a global search for where `CurAccountKey` is provided. This is not a good user experience, and this is when the official Vue documentation recommends using TS.

```
import { inject } from 'vue';
import { AuthAccountKey, CurAccountKey } from '@/constants/injectionKeys';

const curAccount = inject(CurAccountKey);
curAccount.name; // Does curAccount have a name?
```

### Solution
Using TS and InjectionKey can effectively solve the type hinting issue.

`src/types.ts`:

```
export interface Account {
  name: string;
  id: number;
};
```

`src/constants/injection-key.ts`:

```
import { InjectionKey } from 'vue';
import { Account } from '@/types';

export const CurAccountKey: InjectionKey<Account> = Symbol('account')
```

`Main.vue`:

```
import { provide } from 'vue';
import { CurAccountKey } from '@/constants/injectionKeys';

const user = reactive({ id: 1, name: 'youth' });
provide(CurAccountKey, 'name: youth');
provide(CurAccountKey, user);
```

`DeepChild.vue`:

```
const curAccount = inject(CurAccountKey);
curAccount?.age;
curAccount?.id;
```

## Strict Injection

By default, `inject` assumes that the injection name passed in will be provided by some ancestor component in the chain. If indeed no component provides that injection name, it will throw a runtime warning.

```
const curAccount = inject(CurAccountKey);
curAccount?.id;
```

Of course, sometimes we may not require the ancestor chain to provide it. In this case, the official Vue documentation recommends using a default value to solve the situation where the ancestor chain does not provide a value, which only solves the situation where the inject value is not essential.

However, in some cases, we do require the ancestor chain to provide the necessary inject, which is more common in the development of generic components. For example: the `<ElTable>` and `<ElTableColumn>` components, `<ElTableColumn>` must have an `<ElTable>` component in its ancestor chain. It is illegal to use `<ElTableColumn>` alone, and an error ❌ should be thrown instead of a warning ⚠️.

To solve the above strict dependency issue, of course, we can judge in the child component whether the value of inject is undefined and throw an exception if it is. This code is simple:

```
const curAccount = inject(CurAccountKey);
if (!curAccount) {
  throw new Error('CurAccountKey must have a corresponding Provide');
}
curAccount.id;
```

Yes, not bad! It solves the problem! What if there are many strict dependencies? Would we have to use if statements everywhere?

### Solution

Create a strict injection utility function that throws an exception when the corresponding injection name is not provided.

```
export const injectStrict = <T>(key: InjectionKey<T>, defaultValue?: T | (() => T), treatDefaultAsFactory?: false): T => {
  const result = inject(key, defaultValue, treatDefaultAsFactory); 
  if (!result) { 
    throw new Error(`Could not resolve ${key.description}`); 
  } 
  return result;
}
```

Rewrite it using injectStrict:

```
const curAccount = injectStrict(CurAccountKey);
curAccount.id;
```

## Cascaded Penetration

In Vue, the Provide component cannot use the provided value.

This might seem a bit confusing, but in practical terms, the usage is like this:

```
const user = reactive({ id: 1, name: 'youth' });
provide(CurAccountKey, user);
...

inject(CurAccount); // Here it's impossible to obtain the user provided above.
```

Huh? At this point, some might question, using the provide value in the Provide component? Is that a mistake? How could there be such an operation?

```
const user = reactive({ id: 1, name: 'youth' });
provide(CurAccountKey, user);

// When the value of user is needed here, why not use it directly??
user;
```

The issue of cascaded transmission comes up again.

But, don’t forget the scenario with custom hooks!! What if `provide(CurAccountKey, user)`, is inside a custom hook?

`useAccount.ts`:

```
export const useAccount = async () => {
  const user = await fetch('/**/*');
  provide(CurAccountKey, user);
  return { user };
}
```

If it’s a direct call to `useAccount` then it’s not an issue, because `useAccount` returns user. It is very intuitive and convenient to directly destructure `user` at the place where `useAccount` is called.

What if `useAccount` is encapsulated by another hook?

`useApp.ts`:

```
export const useApp = async () => {
  const account = await useAccount();
  ...
  return {
    account
  }
}
```

Of course, this isn’t without a solution, you can destructure account in `useApp` and then return it.

`useApp.ts`:

```
export const useApp = async () => {
  const account = await useAccount();
  ...
  return {
    ...account
  }
}
```

**STOP!!!**

Do you find this situation familiar? If we replace the hook with a component, wouldn’t the situation be just like this:

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Yk3pVzaCllDs0pw9yKaBmg.png)

You can’t just say it’s similar, it’s exactly the same! The emergence of `Provide/Inject` was to solve such issues, but when transparency occurs in hooks, it becomes just like the original problem again!

### Solution

The solution to the above problem is quite simple, which is to obtain the current component instance, and then find the provided value from the component instance!

Since Vue itself does not support the current component obtaining the provide of the current component, let’s implement one ourselves!

```
import { getCurrentInstance, inject, InjectionKey } from 'vue';

export const injectWithSelf = <T>( key: InjectionKey<T>): T | undefined => { 
  const vm = getCurrentInstance() as any; 
  return vm?.provides[key as any] || inject(key);
}
```

Here, we find the provide value with the corresponding key from the current component’s instance. If it doesn’t exist, we use inject to get it from the ancestor chain components.

Let’s rewrite it using injectWithSelf:

`useAccount.ts`:

```
export const useAccount = async () => {
  const user = await fetch('/**/*');
  provide(CurAccountKey, user);
  return { user };
}
```

`useApp.ts`:

```
export const useApp = async () => {
  const account = await useAccount();
  ...
  return {
    account
  }
}
```

`Main.vue`:

```
useApp();

// It must be after useApp()
const user = injectWithSelf(CurAccountKey)
```

## Summary:

- Use Symbol to create injection names to avoid naming conflicts.

- Using TS and InjectionKey can effectively solve type hinting issues.

- Using custom injectStrict can solve strict injection problems.

- Using custom injectWithSelf can solve the issue of return values penetrating through levels when hooks are nested.
