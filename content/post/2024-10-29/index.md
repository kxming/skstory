---
title: 'Usage and Differences of ?., !., and ?? in JavaScript'
date: 2024-10-29T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1372/format:webp/1*6VMwfpFr7bDv_qeLHsk_dA.jpeg"
categories: ["Story","Fiction", "Life"]
tags: ["Story","Fiction", "Life"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

In JavaScript, `?.`, `!.`, and `??` are three different operators, each with its own purpose to enhance code simplicity and safety, especially when dealing with possible null or undefined values. Below is an explanation of each operator:

## Optional Chaining Operator ?.

**Usage**: `obj?.prop` or `obj?.[expr]`

**Function**: When accessing a deeply nested property, if the object `obj` is `null` or `undefined`, the `?.` operator prevents an error from being thrown and instead returns `undefined`, avoiding errors from trying to access non-existent properties.

Example:

```
const user = null;
console.log(user?.name); // Outputs undefined instead of throwing an error
```

## Non-Null Assertion Operator !.

**Usage**: `obj!.prop`

**Function**: This operator tells the TypeScript compiler that you are sure the expression will never be `null` or `undefined`, thus avoiding warnings during type checking. Note that this only affects compile-time type checking and does not change runtime behavior, so if the expression is actually `null` or `undefined`, it will still result in a runtime error.

Example: (Assumes a TypeScript environment)

```
let user: User | null = fetchUserData();
console.log(user!.name); // Tells TypeScript you're sure user is not null or undefined
```

## Nullish Coalescing Operator ??

**Usage**: `value ?? fallback`

**功能**: Returns the `fallback` value if value is `null` or `undefined`; otherwise, it returns `value` itself. This differs from the logical OR `||` operator, which returns `fallback` if `value` is any falsy value (such as 0, false, an empty string, etc.), whereas `??` only replaces `value` when it is strictly `null` or `undefined`.

Example:

```
let num = null;
const result = num ?? 42; // result is 42 because num is null
```

In summary, `?.` helps safely access object properties, `!.` is used to tell the TypeScript compiler to ignore possible `null` or `undefined` checks (ensure safety when using it), and `??` provides an alternative value when the original is `null` or `undefined`.
