---
title: 'Excellent JS Tips and Methods'
date: 2024-08-12T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*06Q7HHp3ppGAdrIFwNVDVg.jpeg"
categories: ["Story","Fiction", "Life"]
tags: ["Story","Fiction", "Life"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

## Array Related

### 1. Declare and Initialize Arrays

You can initialize an array with a specific size using default values like `""`, `null`, or `0`. This is common for one-dimensional arrays, but how about initializing a two-dimensional array/matrix?

```
const array = Array(5).fill('');
// Output: ["", "", "", "", ""]

const matrix = Array(5).fill(0).map(() => Array(5).fill(0));
// Output: [Array(5), Array(5), Array(5), Array(5), Array(5)]
// 0: (5) [0, 0, 0, 0, 0]
// 1: (5) [0, 0, 0, 0, 0]
// 2: (5) [0, 0, 0, 0, 0]
// 3: (5) [0, 0, 0, 0, 0]
// 4: (5) [0, 0, 0, 0, 0]
// length: 5
```

### Filter Out Falsy Values from an Array

Falsy values like `0`，`undefined`，`null`，`false`，`""`，`''` can be easily omitted using:

```
const array = [3, 0, 6, 7, '', false];
array.filter(Boolean);
// Output: [3, 6, 7]
```

### Array Lookup

Use `indexOf()` to find the position of an item in an array. If not found, it returns `-1`. This can be simplified using bitwise NOT (`~`):

Traditional:

```
if(arr.indexOf(item) > -1) {
  ...
}
​
if(arr.indexOf(item) === -1) {
  ...
}
```

Simplified:

```
if(~arr.indexOf(item)) {
  ...
}
​
if(!~arr.indexOf(item)) {
  ...
}
```

Alternatively, use `includes()`:

```
if(arr.includes(item)) {
  ...
}
```

### Shuffle an Array

利用内置`Math.random()`方法。

```
const list = [1, 2, 3, 4, 5, 6, 7, 8, 9];
list.sort(() => {
    return Math.random() - 0.5;
});
// Output:
(9) [2, 5, 1, 6, 9, 8, 4, 3, 7]
// Call it again
(9) [4, 1, 7, 5, 3, 8, 2, 9, 6]
```

### Check for a Safe Array

```
const safeArray = (array) => Array.isArray(array) ? array : [];
```

### 数组清空

Compare `arr.length = 0;` vs `arr = [];`:

The main use of `arr.length = 0` is to completely clear the actual array that the literal points to.

In JavaScript, we know that reference types store addresses in the stack memory, pointing to real data in the heap memory. Suppose we have a non-empty array `arr`.

When we use `arr = []`, we are actually reassigning `arr` to a new empty array `[]`.

At this point, the original array that `arr` was pointing to still exists in memory (which might lead to memory leaks).

Using `arr.length = 0`: This command acts on the real array, effectively clearing the original array directly.

```
// Clear the original array:
arr.length = 0;

// Reassign to a new empty array:
arr = [];
```

### Union, Intersection, and Difference

```
let a = new Set([1, 2, 3]); 
let b = new Set([4, 3, 2]); 
// Union 
let union = new Set([...a, ...b]); // Set {1, 2, 3, 4} 
// Intersection 
let intersect = new Set([...a].filter(x => b.has(x))); // set {2, 3} 
// Difference 
let difference = new Set([...a].filter(x => !b.has(x))); // Set {1}
```

### Map Loop

The `map` method can change the original array if dealing with objects:

```
const cities = [{ name: 'NewYork' }, { name: 'DC' }];
const newCities = cities.map(item => {
  item.country = 'America';
  return item;
});

console.log('cities', cities);
// output
// [{ name: 'NewYork', country: 'America' },{ name: 'DC', country: 'America' }]
console.log('newCities', newCities);// output
// [{ name: 'NewYork', country: 'America' },{ name: 'DC', country: 'America' }]
 ```

## Logical Operations

### Use Logical Operators for Conditions

Reduce nested `if...else` or `switch` with basic logical operators:

```
function doSomething(arg1) {
  arg1 = arg1 || 10;
  // Set arg1 to 10 if not already set
  return arg1;
}

let foo = 10;
foo === 10 && doSomething();
// Output: 10
foo === 5 || doSomething();
// Output: 10
```

### Optional Chaining

Optional chaining `?.` stops evaluation if the value before `?` is undefined or null.

```
const user = {
  employee: {
    name: "Kapil"
  }
};

user.employee?.name;
// Output: "Kapil"
user.employ?.name;
// Output: undefined
```

For dynamic properties:

```
object?.[index]
```

For method calls:

```
object.runsOnlyIfMethodExists?.()
```

### Nullish Coalescing Operator

The `??` operator returns the right operand when the left is null or undefined.

```
const foo = null ?? 'my school';
// Output: "my school"

const baz = 0 ?? 42;
// Output: 0
```

### `||=`and`??=`

`||=` assigns a value only if the left side is falsy, whereas `??=` assigns only if the left side is `null` or `undefined`.

```
let x = null;
x ||= 5;
console.log(x);  // Output: 5

let y = 10;
y ||= 7;
console.log(y);  // Output: 10
let a = null;
a ??= 5;
console.log(a);  // Output: 5
let b = 10;
b ??= 7;
console.log(b);  // Output: 10
```

## Number Related

### Convert Decimal to Binary or Hexadecimal

```
const num = 10;

num.toString(2);
// Output: "1010"
num.toString(16);
// Output: "a"
num.toString(8);
// Output: "12"
```

### Get a Random Integer Between Two Values

```
const random = (min, max) => Math.floor(Math.random() * (max - min + 1) + min);

random(1, 50);
```

### Convert a String to a Number

Conventional:

```
let str = '2'
console.log(Number(str))   //2
```

Optimized:

```
let str = '2'
console.log(~~str)    //2

console.log(+str)    //2
```

### Scientific Notation

Use scientific notation to represent numbers:

```
for (let i = 0; i < 1e7; i++) {}

// All these comparisons return true
1e0 === 1;
1e1 === 10;
1e2 === 100;
1e3 === 1000;
1e4 === 10000;
1e5 === 100000;
```

### Exponentiation

Use `**` for exponentiation:

```
2**3; // 8
2**4; // 16
4**3; // 64
```

### Convert to Boolean

Use double logical NOT to convert to Boolean:

```
!!23 // TRUE
!!"" // FALSE
!!0 // FALSE
!!{} // TRUE
```

## Object Related

### Check if an Object is Empty

```
const isEmpty = obj => Reflect.ownKeys(obj).length === 0 && obj.constructor === Object;
isEmpty({}) // true
isEmpty({a:"not empty"}) //false
```

### Dynamic Property Names

Use brackets for dynamic properties:

```
const key = 'name';
const person = { [key]: 'Alice' };
console.log(person.name); // Output: Alice
```

### Check for a Safe Object

```
const isValidObject = (obj) => typeof obj === 'object' && !Array.isArray(obj) && Object.keys(obj).length;
const safeObject = obj => isValidObject(obj) ? obj : {};

```

## String Related

## Capitalize First Letter

```
const capitalize = str => str.charAt(0).toUpperCase() + str.slice(1);

capitalize("hello world");  // "Hello world"
```

### Reverse a String

```
const reverse = str => str.split('').reverse().join('');

reverse('hello world');   // 'dlrow olleh'
```

### Filter Special Characters

```
function filterCharacter(str) {
  let pattern = new RegExp("[`~!@#$^&*()=：”“'。，、？|{}':;'%,\\[\\].<>/?~！@#￥……&*（）&;—|{ }【】‘；]");
  let resultStr = "";
  for (let i = 0; i < str.length; i++) {
    resultStr = resultStr + str.substr(i, 1).replace(pattern, '');
  }
  return resultStr;
}

// Example
filterCharacter('gyaskjdhy12316789#$%^&!@#1=123,./['); // Result: gyaskjdhy123167891123
```

## Browser Related

### Copy Content to Clipboard

```
const copyToClipboard = (text) => navigator.clipboard.writeText(text);

copyToClipboard("Hello World");
```

Compatibility:

```
const copyText = async (val) => {
  try {
    if (navigator.clipboard && navigator.permissions) {
      await navigator.clipboard.writeText(val);
      return;
    }

    const textArea = document.createElement('textArea');
    textArea.value = val;
    textArea.style.width = 0;
    textArea.style.position = 'fixed';
    textArea.style.left = '-999px';
    textArea.style.top = '10px';
    textArea.setAttribute('readonly', 'readonly');
    document.body.appendChild(textArea);
    textArea.select();

    const success = document.execCommand('copy');
    if (!success) {
      throw new Error('Unable to copy text');
    }
    document.body.removeChild(textArea);
  } catch (err) {
    console.error('Copy failed:', err);
  }
};
```

### Clear All Cookies

```
const clearCookies = document.cookie.split(';').forEach(
  cookie => document.cookie = cookie.replace(/^ +/, '')
  .replace(/=.*/, `=;expires=${new Date(0).toUTCString()};path=/`)
);
```

### Get Selected Text

```
const getSelectedText = () => window.getSelection().toString();
```

### Scroll to the Top of the Page

```
const goToTop = () => window.scrollTo(0, 0);
```

### Check if Scrolled to the Bottom

```
const scrolledToBottom = () => document.documentElement.clientHeight + window.scrollY
                                        >= document.documentElement.scrollHeight;
```

### Check if Tab is Active

```
const isTabInView = () => !document.hidden;
```

### Redirect to a URL

```
const redirect = url => location.href = url

redirect("https://www.google.com/")
```

### Open Print Dialog

```
const showPrintDialog = () => window.print()
```

### Check if Element is Focused

```
const elementIsInFocus = (el) => (el === document.activeElement);
```

## URL Related

### Get Parameters from URL and Convert to Object

网页路径经常是：www.google.com?search=js&xxx=kkk这种形式的，我们是经常需要取参数的，可以使用第三方的qs包实现，如果你只是要实现去参数，这一句代码就可以实现，不用再引入qs了。

```
const getParameters = URL => JSON.parse(`{"${decodeURI(URL.split("?")[1]).replace(/"/g, '\"').replace(/&/g, '","').replace(/=/g, '":"')}"}`);

getParameters("https://www.google.com.hk/search?q=js+md&newwindow=1");
// {q: 'js+md', newwindow: '1'}
```

## Miscellaneous

### Check if a Function

```
const isFunction = (obj) => typeof obj === "function" && typeof obj.nodeType !== "number" && typeof obj.item !== "function";
```

### Debounce/Throttle

**Debounce**: Triggers after a specified time only if no further events are received.

**Throttle**: Triggers once within a specified time frame despite multiple events.

```
// Debounce
function debounce(fn, delay) {
  let timer = null;
  return function () {
    if (timer) {
      clearTimeout(timer);
    }
    timer = setTimeout(() => {
      fn.apply(this);
    }, delay);
  };
}

// Throttle
function throttle(fn) {
  let timer = null;
  return function () {
    if (timer) return;
    timer = setTimeout(() => {
      fn.apply(this, arguments);
      timer = null;
    }, 1000);
  };
}
```

### Common Regex Validations

```
// Validate 2-9 Chinese characters
const validateName = (name) => /^[\u4e00-\u9fa5]{2,9}$/.test(name);

// Validate phone number
const validatePhoneNum = (mobile) => /^1[3-9]\d{9}$/.test(mobile);
// Validate password (6-18 characters: letters, numbers, underscores)
const validatePassword = (password) => /^[a-zA-Z0-9_]{6,18}$/.test(password);
```

## Good Coding Habits

### Use Destructuring

Replace `console.log('name', name)` with `console.log({name})`.

### Clever Use of JS Implicit Type Conversion

Quickly convert to `Number`:

```
const price = +'32';
const timeStamp = +new Date();
```

Quickly convert to `Boolean`:

```
const isTrue = !!'';
```

Quickly convert to `String`:

```
const str = 1 + '';
```

### Use Return Instead of If…Else

Replace:

```
if (condition1) {
  // do condition1 
} else if (condition2) {
  // do condition2
} else if(condition3) {
  // do condition3
}
```

With:

```
if (condition1) {
  // do condition1 
  return;
}

if (condition2) {
  // do condition2
  return;
}

if (condition3) {
  // do condition3  
  return;
}
```

This simplifies code and makes it easier to read and edit.
