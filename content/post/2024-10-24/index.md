---
title: 'Understanding the Differences and Usage of >>>, /deep/, ::v-deep, ::v-deep(), and :deep()'
date: 2024-10-24T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*ryyF1WQUm5ucyfjGRIrIlg.jpeg"
categories: ["Story","Fiction", "Life"]
tags: ["Story","Fiction", "Life"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

In Vue projects, especially when using component-based development, there are times when you need to optimize styles within components. However, Vue's scoped styling feature prevents external styles from directly affecting the internals of a component. To address this, the Vue community introduced deep selectors (also known as piercing or shadow-piercing selectors) that allow you to cross component boundaries and style internal elements.

Deep selectors enable you to penetrate from parent components into child components and directly modify their styles. This is particularly useful when customizing styles for third-party UI library components.

## >>>

`>>>` is a native CSS deep selector syntax used to penetrate style encapsulation.

Compatibility: Only works in certain environments (like specific Webpack css-loader configurations) and native CSS. Typically requires specific configuration in Vue single-file components.
Note: In Vue single-file components, we usually use CSS preprocessors. However, preprocessors like Sass cannot correctly parse `>>>,` so it's not recommended. Instead, use /deep/ or ::v-deep, which are aliases and work correctly.

```
<style scoped>
.parent >>> .child {
  ...
}
</style>
```

## /deep/

`/deep/`was once proposed in CSS but was later removed, so it's not recommended.

- **Compatibility**: Supports CSS preprocessors (like Sass, Less) and native CSS.

- **Note**: In Vue 3, `/deep/` is no longer officially supported. Some build tools or libraries might still be compatible, but its use will generate warnings during compilation.

> `/deep/` usage as a combinator has been deprecated. Use `:deep()` instead.

```
<style scoped>
.parent /deep/ .child {
  ...
}
</style>
```

## ::v-deep
  
`::v-deep`is an alias for the `/deep/` deep selector.

- **Compatibility**: Supported in Vue 2, but not recommended in Vue 3.

- **Note**: In Vue 3, `::v-deep` is also no longer officially supported. Its use will generate warnings during compilation.

> `::v-deep` usage as a combinator has been deprecated. Use :deep() instead.

```
<style scoped>
.parent::v-deep .child {
  ...
}
</style>
```

## ::v-deep()

`::v-deep()`is a transitional combinator from Vue 2 to Vue 3.

**Usage**: Supported in Vue 3, but considered deprecated and will trigger warnings during compilation.

```
<style scoped>
.parent ::v-deep(.child) {
  ...
}
</style>
```

## :deep()
  
`:deep()`is the officially recommended deep selector in Vue 3. The use of `>>>`, `/deep/`, `::v-deep`, and `::v-deep()` is discouraged.

- **Compatibility**: Supported in Vue 3, not usable in Vue 2.

- **Documentation**: [Vue.js SFC CSS Features]((https://cn.vuejs.org/api/sfc-css-features#style-scoped))

```
<style scoped>
.parent :deep(.child) {
  ...
}
</style>
```

## Evolution
  
Sometimes we may want to explicitly make a rule target child components.

Originally we supported the >>> combinator to make the selector "deep". However, some CSS pre-processors such as SASS has issues parsing it since this is not an official CSS combinator.

We later switched to /deep/, which was once an actual proposed addition to CSS (and even natively shipped in Chrome), but later dropped. This caused confusion for some users, since they worry that using /deep/ in Vue SFCs would make their code not supported in browsers that have dropped the feature. However, just like >>>, /deep/ is only used as a compile-time hint by Vue's SFC compiler to rewrite the selector, and is removed in the final CSS.

To avoid the confusion of the dropped /deep/ combinator, we introduced yet another custom combinator, ::v-deep, this time being more explicit that this is a Vue-specific extension, and using the pseudo-element syntax so that any pre-processor should be able to parse it.

The previous versions of the deep combinator are still supported for compatibility reasons in the current Vue 2 SFC compiler, which again, can be confusing to users. In v3, we are deprecating the support for >>> and /deep/.

As we were working on the new SFC compiler for v3, we noticed that CSS pseudo elements are in fact semantically NOT combinators. It is more consistent with idiomatic CSS for pseudo elements to accept arguments instead, so we are also making ::v-deep() work that way. The current usage of ::v-deep as a combinator is still supported, however it is considered deprecated and will raise a warning.

以上内容来自：https://github.com/vuejs/rfcs/blob/scoped-styles-changes/active-rfcs/0023-scoped-styles-changes.md#deep-selectors

`>>>` → `/deep/` → `::v-deep` → `::v-deep()` → `:deep()`

The evolution trend of deep selectors is evident:

**Combinator Selector** → **Pseudo-element** → **Pseudo-class**

## Conclusion

- Use `::v-deep` in Vue 2.

- Use `:deep()` in Vue 3.

- `/deep/` requires specific browser versions and is not recommended.

- Some CSS preprocessors poorly support `>>>`. Use it only without preprocessors; otherwise, avoid it.
