---
title: 'The forEach You Didn’t Know About'
date: 2024-06-14T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*-fUNketACA0rEYNRnZNz4A.jpeg"
categories: ["Code","Javascript"]
tags: ["Code","Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "forEach() is a prototype method of an array object, which executes a given callback function once for each element in the array, and always returns undefined."
---

## Overview

`forEach()` is a prototype method of an array object, which executes a given callback function once for each element in the array, and always returns undefined. Array-like objects do not have a `forEach` method, such as arguments. The usage of `forEach` is relatively simple:

```
arr.forEach(callback(currentValue [, index [, array]])[, thisArg])
```

### Parameter explanation:

1. `callback`: The callback function to be executed for every element in the array, can have 1–3 parameters

  - `currentValue`: The current element being processed, **required**

  - `index`: The index of the current element being processed, **optional**

  - `array`: The original array object that the forEach method operates on, **optional**

2. `thisArg`: The this context of the callback function during execution, defaults to the global object, **optional**

## Note

### First: Does not support handling asynchronous functions

The `forEach()` method in JavaScript is a synchronous method and does not support handling asynchronous functions. If you execute an asynchronous function within `forEach`, `forEach()` does not wait for the asynchronous function to complete; it proceeds to the next item. This means that if you use asynchronous functions within `forEach()`, the execution order of asynchronous tasks cannot be guaranteed.

```
async function test() {
    let arr = [3, 2, 1]
    arr.forEach(async item => {
        const res = await mockSync(item)
        console.log(res)
    })
    console.log('end')
}

function mockSync(x) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
                resolve(x)
        }, 1000 * x)
    })
}
test()
// expectation:
// 3
// 2 
// 1
// end

// in fact:
// end
// 1
// 2
// 3
```

Solve

1. Method One

You can use methods such as `map()`, `filter()`, `reduce()`, etc., which support returning a Promise in the function and will wait for all Promises to complete.

Here is an example code of using `map()` and `Promise.all()` to handle asynchronous functions:

```
const arr = [1, 2, 3, 4, 5];

async function asyncFunction(num) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(num * 2);
    }, 1000);
  });
}

const promises = arr.map(async (num) => {
  const result = await asyncFunction(num);
  return result;
});

Promise.all(promises).then((results) => {
  console.log(results); // [2, 4, 6, 8, 10]
});
```

Since we used the await keyword in the asynchronous function, the `map()` method will wait for the asynchronous function to complete and return a result, so we can correctly handle asynchronous functions.

2. Method Two

Using a for loop to handle asynchronous functions.

```
const arr = [1, 2, 3, 4, 5];

async function asyncFunction(num) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(num * 2);
    }, 1000);
  });
}

async function processArray() {
  const results = [];
  for (let i = 0; i < arr.length; i++) {
    const result = await asyncFunction(arr[i]);
    results.push(result);
  }
  console.log(results); // [2, 4, 6, 8, 10]
}

processArray();
```

### Second: Cannot catch errors in asynchronous functions

If an asynchronous function throws an error during execution, `forEach()` cannot catch that error. This means that even if an error occurs in the asynchronous function, `forEach()` will still proceed.

### Third: Apart from throwing an exception, there is no way to terminate or break out of a forEach() loop

The `forEach()` method does not support using break or continue statements to jump out of the loop or skip an item. If you need to break out of the loop or skip an item, you should use a for loop or other methods that support break or continue statements.

```
let arr = [1, 2, 3]
try {
 arr.forEach(item => {
  if (item === 2) {
   throw('error')
  }
  console.log(item)
 })
} catch(e) {
 console.log('e: ', e)
}

// 1
// e: error
```

### Fourth: Deleting its own element, the index cannot be reset

In forEach, we can’t control the value of the index; it just increments blindly until it exceeds the length of the array and exits the loop. Therefore, it’s also impossible to delete itself to reset the index. Let’s look at a simple example.

```
let arr = [1,2,3,4]
arr.forEach((item, index) => {
    console.log(item); // 1 2 3 4
    index++;
});
```

### Fifth: Issue with the ‘this’ context

In the `forEach()` method, the **this** keyword refers to the object that calls the method. However, when using a regular function or an arrow function as the parameter, the scope of the **this** keyword might face issues. In arrow functions, the **this** keyword refers to the object in which the function was defined. In regular functions, the **this** keyword refers to the object that calls the function. To ensure the correct scope for the **this** keyword, the `bind()` method can be used to bind the function’s scope. Here’s an example illustrating the issue of the **this** keyword scope in the `forEach()` method:

```
const obj = {
  name: "Alice",
  friends: ["Bob", "Charlie", "Dave"],
  printFriends: function () {
    this.friends.forEach(function (friend) {
      console.log(this.name + " is friends with " + friend);
    });
  },
};
obj.printFriends();
```

In this example, we defined an object named obj with a printFriends() method. Inside the printFriends() method, we use the forEach() method to iterate over the friends array and use a regular function to print each friend's name and obj object's name property. However, upon running this code, the output will show:

```
undefined is friends with Charlie
undefined is friends with Dave
```

This is because, within the `forEach()` method when using a regular function, the scope of the function is not the object calling the `printFriends()` method but the global scope. Hence, the obj object's properties are not accessible within that function.

To solve this problem, the `bind()` method can be used to bind the function’s scope, or an arrow function can be used to define the callback function. Here’s an example of solving the issue using the `bind()` method:

```
const obj = {
  name: "Alice",
  friends: ["Bob", "Charlie", "Dave"],
  printFriends: function () {
    this.friends.forEach(
      function (friend) {
        console.log(this.name + " is friends with " + friend);
      }.bind(this) // Using the bind() method to bind the function's scope
    );
  },
};
obj.printFriends();
```

In this example, we use the bind() method to bind the function’s scope, tying its scope to the obj object. After running the code, the output is:

```
Alice is friends with Charlie
Alice is friends with Dave
```

By binding the function’s scope using the `bind()` method, we can access the properties of the obj object correctly.

Another solution is to use an arrow function. Since an arrow function does not have its own **this**, it inherits the **this** from its current scope. Hence, in an arrow function, the **this** keyword refers to the object in which the function was defined. The code is omitted for brevity.

### Sixth: forEach has lower performance than for loop

- `for`: The for loop doesn’t have extra function call stacks and context, so its implementation is the simplest.

- `forEach`: For forEach, its function signature includes parameters and context, so its performance is lower than the for loop.

### Seventh: Skips over deleted or uninitialized items

```
// Skipping uninitialized values
const array = [1, 2, /* empty */, 4];
let num = 0;

array.forEach((ele) => {
  console.log(ele);
  num++;
});

console.log("num:",num);

//  1
//  2 
//  4 
// num: 3

// Skipping deleted values
const words = ['one', 'two', 'three', 'four'];
words.forEach((word) => {
  console.log(word);
  if (word === 'two') {
      // When reaching the item containing the value 'two', the entire array's first item was removed
      // This causes all remaining items to move up one position. Because the element 'four' is now at a more forward position in the array, 'three' will be skipped.
      words.shift(); // 'one' will be removed from the array
  }
}); // one // two // four

console.log(words); // ['two', 'three', 'four']
```

### Eighth: Does not change the original array

When `forEach()` is called, it does not change the original array, which is the array it was called on. However, the object may be changed by the callback function passed in.

```
// Example 1
const array = [1, 2, 3, 4]; 
array.forEach(ele => { ele = ele * 3 }) 
console.log(array); // [1,2,3,4]

// Solution, to change the original array
const numArr = [33,4,55];
numArr.forEach((ele, index, arr) => {
    if (ele === 33) {
        arr[index] = 999
    }
})
console.log(numArr);  // [999, 4, 55]

// Example 2
const changeItemArr = [{
    name: 'wxw',
    age: 22
}, {
    name: 'wxw2',
    age: 33
}]
changeItemArr.forEach(ele => {
    if (ele.name === 'wxw2') {
        ele = {
            name: 'change',
            age: 77
        }
    }
})
console.log(changeItemArr); // [{name: "wxw", age: 22},{name: "wxw2", age: 33}]

// Solution
const allChangeArr = [{
    name: 'wxw',
    age: 22
}, {
    name: 'wxw2',
    age: 33
}]
allChangeArr.forEach((ele, index, arr) => {
    if (ele.name === 'wxw2') {
        arr[index] = {
            name: 'change',
            age: 77
        }
    }
})
console.log(allChangeArr); // [{name: "wxw", age: 22},{name: "change", age: 77}]
```
