---
title: 'Understanding Fragment in Vue3'
date: 2024-02-24T16:22:38+08:00
draft: true
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*YMMs8s-H6fct1k8DJ1vLRw.png"
categories: ["Javascript","Vue"]
tags: ["Javascript","Vue"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

I wonder if anyone has encountered the following error information in Vue2:

```
Errors compiling template:
Component template should contain exactly one root element. If you are using v-if on multiple elements, use v-else-if to chain them instead.
```

This is an error prompt thrown by Vue2. It means that a component can only have one root element. When we create a new Vue page, there are usually multiple different element nodes. We will wrap a div at the outermost layer to make it the root node of this page. But this is not user-friendly. Sometimes we don’t need this div element.

Vue3 has solved this problem. Vue3 has introduced a new DOM-like tag element `<Fragment></Fragment>`. If there are multiple element nodes in the Vue page, Vue will add a `<Fragment></Fragment>` tag to these element nodes during compilation. And this tag does not appear in the DOM tree.

Example

```
<template>
 <header>...</header>
 <main v-bind="$attrs">...</main>
 <footer>...</footer>
</template>
```
