---
title: "I Feel Like I Haven't Learned the split Method Properly"
date: 2024-10-28T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1372/format:webp/1*LlqGR3DjTu_f9ApsWxZ2lQ.jpeg"
categories: ["Story","Fiction", "Life"]
tags: ["Story","Fiction", "Life"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

Friends, can you use the `split` method in JavaScript? I recently realized that I haven't been using it correctly for years, which caused me to get stuck on a problem while solving challenges. Today, I must document this.

Before discussing the problem, let's review the two basic uses of `split`.

## Character Parameter

The `split` method is used for string splitting, and the most common form is using a specific character to split the original string. For example:

```
'as:st:vii'.split(':') 
// ['as', 'st', 'vii']
```

Using `:` to split the original string results in an array. This is the usage most people are familiar with.

## Regex Parameter

Sometimes splitting isn't done by a specific character but by a class of characters, like numbers:

```
'as2st3vii'.split(/\d/)
// ['as', 'st', 'vii']
```

The regex `\d` represents all digits, treating any digit as a separator, which is very flexible.

## Problem One

So why do I say I haven't learned it properly? Look at the following code:

```
`asabstcdvii`.split(/(ab|cd)/)
```

Think about the result. If you think it returns `['as', 'st', 'vii']`, you should reconsider.

```
// ['as', 'ab', 'st', 'cd', 'vii']
```

This is the correct result, including the separators `ab` and `cd` in the output. If you got it right, congratulations. If not, remember that [MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/split) is a great resource.

> If the separator is a regular expression with **capturing groups**, then each time the separator matches, the captured groups (including any undefined results) are spliced into the output array.

To explain simply, if there are **capturing groups** in the regex, their contents will also be included in the result array.

If we modify the regex to be a bit more complex:

```
`asabstcdvii`.split(/(a|c)(b|d)/)
// ['as', 'a', 'b', 'st', 'c', 'd', 'vii']
```

You can see that all **capturing groups** in the regex are returned in the result array.

What is this feature useful for? I haven't found a real-world application since I didn't know about it before. Here's a contrived example:

```
function addYear(date) {
  const arr = date.split(/(-|\/)/);
  arr[0] = Number(arr[0]) + 1;
  return arr.join('');
}
```

If the date string separator could be `-` or `/`, using **capturing groups** allows you to easily identify the original separator after splitting.

## Problem Two

Wait, there's more. The `split` method also supports a second parameter, `limit`, which restricts the length of the resulting array.

```
`asabstcdvii`.split(/(a|c)(b|d)/, 2);
// ['as', 'a']
```

When this parameter is provided, the split method will split the string at each occurrence of the separator, but will stop once there are limit elements. Any remaining text won't be included in the array.
