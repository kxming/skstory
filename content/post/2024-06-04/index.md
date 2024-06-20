---
title: '4 Methods to Determine If the Contents of Two Arrays Are Equal'
date: 2024-06-04T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Zm-09l_F3UwVf2IneHGHew.jpeg"
categories: ["Code","Javascript"]
tags: ["Code","Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "When I was a child, I ate many foods, and now I can’t remember what I had eaten. But one thing is for sure, some of them have become part of my bones and flesh. The transformation that reading brings to a person is the same."
---

Given two arrays, how to determine if the content of both arrays is equal without using sorting and without considering the position of elements.

Example:

```
[1, 2, 3] and [1, 3, 2] // true
[1, 2, 3] and [1, 2, 4] // false
```

## 1. Count Elements Frequency

```
function areArraysContentEqual(arr1, arr2) {
  if (arr1.length !== arr2.length) {
    return false;
  }

  // Create a counter object to record the frequency of each element in the array
  const countMap1 = count(arr1)
  const countMap2 = count(arr2)

  // Count the frequency of elements in the array
  function count(arr = []) {
    const resMap = new Map();
    for (const item of arr) {
      resMap.set(item, (resMap.get(item) || 0) + 1);
    }
    return resMap
  }
  // Check if the counter objects are equal
  for (const [key, count] of countMap1) {
    if (countMap2.get(key) !== count) {
      return false;
    }
  }

  return true;
}

const array1 = ["apple", "banana", "cherry", "banana", 1, '1', '11', 11];
const array2 = ["banana", "apple", "banana", "cherry", '1', 1, '11', 11];

areArraysContentEqual(array1, array2) // true
```

## 2. One Object

```
function areArraysContentEqual3(arr1= [], arr2 = []) {
  if (arr1.length !== arr2.length) {
    return false;
  }

  const countMap = new Map();

  // Count the elements of the first array
  for (const item of arr1) {
    countMap.set(item, (countMap.get(item) || 0) + 1);
  }

  // Compare the second array with the count
  for (const item of arr2) {
    const val = countMap.get(item);
    if (val === undefined || val <= 0) {
      return false;
    }
    countMap.set(item, val - 1);
  }

  return true;
}
```

## 3. Manipulating the Second Array

```
function areArraysContentEqual(arr1=[], arr2=[]) {
  arr2 = [...arr2]
  if (arr1.length !== arr2.length) {
    return false;
  }

  const compare = (item1, item2) => {
    if (Number.isNaN(item1) && Number.isNaN(item2)) {
      return true;
    }
    return item1 === item2;
  };

  arr1.some(item => {
    // Finding the position of an element in the second array
    const index = arr2.findIndex(item1 => compare(item, item1))
    if (index !== -1 ) {
      arr2.splice(index, 1)
    }
  })
  return !arr2.length
}
```

## 4. Count + turn string

```
function isArrSame(arr1, arr2) {
  if (arr1.length !== arr2.length) return false;
  
  const set = [...new Set([...arr1, ...arr2])]
  function getCounts(arr) {
    return set.map(item => arr.filter(ele => [ele].includes(item)).length).join('')
  }
  return getCounts(arr1) === getCounts(arr2)
}
```
