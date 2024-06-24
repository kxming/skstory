---
title: 'Advanced Usage of Reduce You Might Not Know'
date: 2024-06-23T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*h7QTz6KqyeYG_ausGQjj5g.jpeg"
categories: ["Code","Javascript"]
tags: ["Code","Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "In JavaScript, reduce is a powerful array method that processes each element in an array one by one and combines them into a single value..."
---

In JavaScript, reduce is a powerful array method that processes each element in an array one by one and combines them into a single value. It can be used in various scenarios, from calculating sums to transforming data, making it very useful.

## Basic Syntax

The basic syntax of the reduce method is as follows:

```
array.reduce(function(total, currentValue, currentIndex, arr), initialValue)
```

Here, array is the array to be operated on, callback is a callback function for processing each array element, and initialValue is an optional initial value. The specific parameter descriptions are as follows:

- `function(total, currentValue, currentIndex, arr)` — **Required**. The function to execute for each array element.

  - `total` — **Required**. The initial value, or the return value after computation.

  - `currentValue` — **Required**. The current element.

  - `currentIndex` — **Optional**. The index of the current element.

  - `arr` — **Optional**. The array object to which the current element belongs.

- `initialValue` — **Optional**. The initial value passed to the function.

### Process

1. Use `total` as the initial value of the accumulated result. If totalis not set, the first element of the array is used as the `initialValue`.

2. Start iterating, process `currentValue` with the accumulator, accumulate the mapping result of `currentValue` onto `total`, end this loop, and return `total`.

3. Enter the next loop, repeat the above operation, until the last element of the array.

4. End the iteration and return the final `total`.

### Usage Examples

1. Calculating the Sum of an Array

```
const numbers = [1, 2, 3, 4, 5];
const sum = numbers.reduce((total, currentValue) => total + currentValue, 0);
console.log(sum); // 15
```

2. Transforming Array Elements

```
const words = ["Hello", "world", "this", "is", "reduce"];
const concatenated = words.reduce((total, currentValue) => total + " " + currentValue);
console.log(concatenated); // "Hello world this is reduce"
```

3. Finding the Maximum Value

```
const values = [10, 5, 8, 20, 3];
const max = values.reduce((total, currentValue) => Math.max(total, currentValue), -Infinity);
console.log(max); // 20
```

## Advanced Usage

### Flatten Array

```
function Flat(arr = []) {
    return arr.reduce((total, currentValue) => total.concat(Array.isArray(currentValue) ? Flat(currentValue) : currentValue), [])
}

const arr = [0, 1, [2, 3], [4, 5, [6, 7]], [8, [9, 10, [11, 12]]]];
Flat(arr); // [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 12]
```

### Split Array

```
function Chunk(arr, size) {
    return arr.length ? arr.reduce((total, currentValue) => (total[total.length - 1].length === size ? total.push([currentValue]) : total[total.length - 1].push(currentValue), total), [[]]) : [];
}

const arr = [1, 2, 3, 4, 5];
Chunk(arr, 2); // [[1, 2], [3, 4], [5]]
```

### Counting Occurrences

```
function Counting(arr) {
  return arr.reduce((total, currentValue) => {
    total[currentValue] = (total[currentValue] || 0) + 1;
    return total;
  }, {})
}

const fruits = ['apple', 'banana', 'apple', 'orange', 'banana', 'apple'];

const fruitCounts = fruits.reduce((total, currentValue) => {
  total[currentValue] = (total[currentValue] || 0) + 1;
  return total;
}, {});

console.log(fruitCounts);
/*
{
  'apple': 3,
  'banana': 2,
  'orange': 1
}
*/
```

### Tracking Positions

```
function Position(arr, val) {
  return arr.reduce((total, currentValue, currentIndex) => {
    if (currentValue === val) {
      total.push(currentIndex)
    }
    return total
  }, []);
}

const arr = [2, 1, 5, 4, 2, 1, 6, 6, 7];
Position(arr, 2); // [0, 4]
```

### Combination Function

```
const add5 = (x) => x + 5;
const multiply3 = (x) => x * 3;
const subtract2 = (x) => x - 2;

const composedFunctions = [add5, multiply3, subtract2];

const result = composedFunctions.reduce((total, currentValue) => currentValue(total), 10);
console.log(result); // 43
```

### Array Deduplication

```
function Uniq(arr) {
    return arr.reduce((total, currentValue) => total.includes(currentValue) ? total : [...total, currentValue], []);
}

const arr = [2, 1, 0, 3, 2, 1, 2];
Uniq(arr); // [2, 1, 0, 3]
```

### Calculate the Average Value

```
function Average(arr) {
  return arr.reduce((total, currentValue, currentIndex, array) => {
    total += currentValue;
    if (currentIndex === array.length - 1) {
      return total / array.length;
    }
    return total;
  }, 0)
}

const grades = [85, 90, 92, 88, 95];
console.log(Average(grades)); // Output: 90
```

### Array Max and Min Values

```
function Max(arr = []) {
    return arr.reduce((total, currentValue) => total > currentValue ? total : currentValue);
}

function Min(arr = []) {
    return arr.reduce((total, currentValue) => total < currentValue ? total : currentValue);
}

const arr = [12, 45, 21, 65, 38, 76, 108, 43];
Max(arr); // 108
Min(arr); // 12
```

### URL Parameter Deserialization

```
function ParseUrlSearch() {
    return str.replace(/(^\?)|(&$)/g, "").split("&").reduce((total, currentValue) => {
        const [key, val] = currentValue.split("=");
        total[key] = decodeURIComponent(val);
        return total;
    }, {});
}

const str = 'key1=value1&key2=value2&key3=value3';
console.log(ParseUrlSearch(str)); 
// { key1: 'value1', key2: 'value2', key3: 'value3' }
```

### URL Parameter Serialization

```
function StringifyUrlSearch(search) {
    return Object.entries(search).reduce(
        (total, currentValue) => `${total}${currentValue[0]}=${encodeURIComponent(currentValue[1])}&`,
        Object.keys(search).length ? "?" : ""
    ).replace(/&$/, "");
}

const params = { foo: "bar", baz: 42 };
console.log(StringifyUrlSearch(params)); // "?foo=bar&baz=42"
```

### Grouping Objects

```
function Grouping(arr, key) {
  return arr.reduce((total, currentValue) => {
    if (!total[currentValue[key]]) {
      total[currentValue[key]] = [];
    }
    total[currentValue[key]].push(currentValue);
    return total;
  }, {});
}

const people = [
  { name: 'Alice', age: 25 },
  { name: 'Bob', age: 30 },
  { name: 'Charlie', age: 25 },
  { name: 'Dave', age: 30 }
];
console.log(Grouping(people, 'age'));
/*
{
  '25': [{ name: 'Alice', age: 25 }, { name: 'Charlie', age: 25 }],
  '30': [{ name: 'Bob', age: 30 }, { name: 'Dave', age: 30 }]
}
*/
```

### Create Lookup Map
Use `reduce()` to create a lookup map from an array. After mapping, the time complexity becomes O(1).

```
function arrayToMap(arr, key) {
  return arr.reduce((total, currentValue) => {
    total[currentValue[key]] = currentValue;
    return total;
}, {})
}

const products = [
  { id: 1, name: 'Laptop', price: 999 },
  { id: 2, name: 'Phone', price: 699 },
  { id: 3, name: 'Tablet', price: 499 },
];
const productMap = arrayToMap(product, 'id');

console.log(productMap);
/*
{
  '1': { id: 1, name: 'Laptop', price: 999 },
  '2': { id: 2, name: 'Phone', price: 699 },
  '3': { id: 3, name: 'Tablet', price: 499 }
}
*/
// Accessing a product by ID
const laptop: Product = productMap[1];
console.log(laptop); // { id: 1, name: 'Laptop', price: 999 }
```

### Merge Multiple Arrays into One Object

```
function Merge(arr1, arr2) {
  return arr1.reduce((total, currentValue, index) => {
    total[currentValue] = arr2[index];
    return total;
  }, {})
}

const keys = ['name', 'age', 'gender'];
const values = ['Alice', 25, 'female'];
console.log(person); // { name: 'Alice', age: 25, gender: 'female' }
```

### Check if a String is a Palindrome

```
function isPalindrome(str) {
  return str.split('').reduce((total, currentValue, index, array) => {
    return total && currentValue === array[array.length - index - 1];
  }, true)
}

const str = 'racecar';
console.log(isPalindrome(str)); // Output: true
```
