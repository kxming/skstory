---
title: 'What new features have been added to ES7,8,9,10,11,12?'
date: 2024-02-11T16:22:38+08:00
draft: true
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Wyu4L8Y0ciJ76ZkoP86gqw.jpeg"
categories: ["Javascript"]
tags: ["Javascript"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

## ES7

The following features have been added to ES2016 (ES7)

### Array.prototype.includes

`includes()` method is used to determine whether an array or string contains a specified value

Return value: `true` if included, `false` otherwise.

**Syntax**

- `arr.includes(valueToFind)`

- `arr.includes(valueToFind, fromIndex)`

```
let arr = [1, 2, 3, 4];
arr.includes(3);        // true
arr.includes(5);        // false
arr.includes(3, 1);     // true
```

`fromIndex` is greater than or equal to the array length

Return `false`

```
arr.includes(3, 3);     // false
arr.includes(3, 20);    // false
```

The calculated index is less than 0

If `fromIndex` is negative, use the index calculated by arrlength + fromIndex as the new fromIndex . If the new fromIndex is negative, then Search the entire array.

```
arr.includes(3, -100);  // true
arr.includes(3, -1);    // false
```

### Exponentiation Operator Power operation

Power operator ** , equivalent to Math.pow()

```
5 ** 2                  // 25
Math.pow(5, 2)          // 25
```

## ES8

ES2017 (ES8) has added the following features

- `Async functions`

- `Object.entries`

- `Object.values`

- `Object.getOwnPropertyDescriptors`

- `Trailing commas`

### Async functions

`Async functions` is a function declared by `async` and the `async` function is an instance of the `AsyncFunction` constructor, where `await` is allowed Keywords.

Return value: a Promise

**Syntax**

```
async function name([param[, param[, ...param]]]) {
  // statements
}
```

Example:

```
const promise = () => {
  console.log('1');
  return new Promise((resolve, reject) => {
    resolve('2');
  });
};

const asyncFun = async() => {
  console.log('3');
  const test = await promise();
  console.log('4', test);
}

asyncFun();                        // 3 1 4 2
```

### Object.entries

Return value: The Object.entries() method returns an array of key-value pairs for the given object's own enumerable properties.

**Syntax**

```
Object.entries(obj);
```

Example:

```
let obj = {a: 1, b: 2};
Object.entries(obj);        // [['a', 1], ['b', 2]]
```

### Object.values

Return value: The Object.values() method returns an array of the enumerable property values ​​of the given object itself.

**Syntax**

```
Object.values(obj);
```

Example:

```
let obj = {a: 1, b: 2};
Object.values(obj);         // [1, 2]
```

### Object.getOwnPropertyDescriptors

Return value: The Object.getOwnPropertyDescriptors() method is used to obtain the descriptors of all its own properties of an object.

**Syntax**

```
Object.getOwnPropertyDescriptors(obj);
```

Example:

```
let obj = {a: 1, b: 2};
Object.getOwnPropertyDescriptors(obj);   // [a: {configurable: true, enumerable: true, value: 1, writable: true}, b: {configurable: true, enumerable: true, value: 2, writable: true}]
```

### Trailing commas

If you want to add a new attribute, and the previous line already used a trailing comma, you can just add the new line without modifying the previous line.

JSON does not allow trailing commas

Example:

- Trailing comma in literal

```
// Object
let obj = {
  a: 1,
  b: 2
}

// Array
let arr = [
  1, 
  2
]
```

- Trailing comma in function

```
// Parameter definition
function(x, y) {}
function(x, y,) {}
(x, y) => {}
(x, y,) => {}

// function call
fun(x, y)
fun(x, y,)

// Illegal trailing comma
// Containing no parameters or adding commas after the remaining parameters are illegal trailing commas.
function(,) {}
(,) => {}
fn(,)
function(...arg,) {}
(...arg,) => {}
```

- Trailing comma in deconstruction

```
let [a, b,] = [1, 2];
let {x, y} = {x: 1, y: 2};
```

### String.prototype.padStart()

`padStart()` Fills the current string with another string.

Return value: A new string formed by filling the specified padding string at the beginning of the original string until the target length.

**Syntax**

```
str.padStart(targetLength);
str.padStart(targetLength, padString);
```

- **targetLength**: The target length to which the current string needs to be filled. If this value is less than the length of the current string, the current string itself is returned.

- **padString** (optional): padding string. If the string is too long and the padded string length exceeds the target length, only the leftmost part is retained and the other parts are truncated. The default value for this parameter is “ “.

Example:

```
'abc'.padStart(10);         // "       abc"
'abc'.padStart(10, "foo");  // "foofoofabc"
'abc'.padStart(6,"123465"); // "123abc"
'abc'.padStart(8, "0");     // "00000abc"
'abc'.padStart(1);          // "abc"
```

### String.prototype.padEnd()

The `padEnd()` method fills the current string with a string (repeatedly if necessary).

Return value: Returns a new string formed by filling the specified padding string at the end of the original string until the target length.

**Syntax**

```
str.padEnd(targetLength)
str.padEnd(targetLength, padString)
```

- **targetLength**: The target length to which the current string needs to be filled. If this value is less than the length of the current string, the current string itself is returned.

- **padString** (optional): padding string. If the string is too long and the padded string length exceeds the target length, only the leftmost part is retained and the other parts are truncated. The default value for this parameter is “ “.

Example:

```
'abc'.padEnd(10);          // "abc       "
'abc'.padEnd(10, "foo");   // "abcfoofoof"
'abc'.padEnd(6, "123456"); // "abc123"
'abc'.padEnd(1);           // "abc"
```

## ES9

ES2018 (ES9) has added the following features

- Async iterators

- Object rest properties

- Object spread properties

- Promise.prototype.finally

### Async iterators

Return value: **The next()** method of Async iterator object returns a Promise . The return value of this Promise can be parsed into the format of {value, done}

**Syntax**

```
iterator.next().then(({value, done}) => {});
```

Example:

```
const asyncIterator = () => {
  const array = [1, 2];
  return {
    next: function() {
      if(array.length) {
        return Promise.resolve({
          value: array.shift(),
          done: false
        });
      }
      return Promise.resolve({
        done: true
      });
    }
  }
}

let iterator = asyncIterator();

const test = async() => {
  await iterator.next().then(console.log);  // {value: 1, done: false}
  await iterator.next().then(console.log);  // {value: 2, done: false}
  await iterator.next().then(console.log);  // {done: true}
}

test();
```

You can use **for-await-of** to call functions asynchronously in a loop

```
const promises = [
  new Promise((resolve) => resolve(1)),
  new Promise((resolve) => resolve(2)),
  new Promise((resolve) => resolve(3)),
];

const test = async() => {
  for await (const p of promises) {
    console.log('p', p);
  }
};

test();
```

### Object rest properties

Example:

```
let test = {
  a: 1,
  b: 2,
  c: 3,
  d: 4
}

let {a, b, ...rest} = test;

console.log(a);               // 1
console.log(b);               // 2
console.log(rest);            // {c: 3, d: 4}
```

Note: null Cannot use spread operator

### Object spread properties

Example:

```
let test = {
  a: 1,
  b: 2
}
let result = {c: 3, ...test};
console.log(result);             // {c: 3, a: 1, b: 2}

let test = null;
let result = {c: 3, ...test};    // {c: 3}
```

### Promise.prototype.finally

At the end of Promise , whether the result is resolved or rejected , the method in finally will be called

The callback function in finally does not accept any parameters

Return value: a Promise

**Syntax**

```
const promise = new Promise((resolve, reject) => {
  resolve('resolved');
  reject('rejectd');
})

promise.then((res) => {
  console.log(res);
}).finally(() => {
  console.log('finally')
});
```

Example:

```
const promise = new Promise((resolve, reject) => {
  resolve(1);
  reject(2);
});

const test = () => {
  console.log(3);
  promise.then((res) => {
    console.log(4, res);
  }).catch(err => {
    console.log(5, err);
  }).finally(() => {
    console.log(6);
  });
};

test();  // 3 4 1 6
```

## ES10

ES2019 (ES10) has added the following new features

- Array.prototype.{flat, flatMap}Flatten nested arrays

- Object.fromEntries

- String.prototype.{trimStart, trimEnd}

- Symbol.prototype.description

- Optional catch binding

- Array.prototype.sort() is now required to be stable

### Array.prototype.{flat, flatMap} Flatten nested arrays

**Array.prototype.flat**

The `flat()` method will traverse the recursive array according to a specifiable depth, and merge all elements with the elements in the traversed sub-array into a new array and return it.

Return value: a new array, the old array will not be changed.

**Syntax**

```
arr.flat([depth]);
```

Example:

```
const arr = [1, 2, [[[[3, 4]]]]];

arr.flat();          // [1, 2, [[[3, 4]]]]
arr.flat(3);         // [1, 2, [3, 4]]
arr.flat(-1);        // [1, 2, [[[[3, 4]]]]]
arr.flat(Infinity);  // [1, 2, 3, 4]
```

Note: `flat()` will remove empty items from the array

```
let arr = [1, 2, , , 3];
arr.flat();           // [1, 2, 3]
```

**Implement flat**

```
function customFlat(arr, depth = 1) {
    if(!Array.isArray(arr) || depth <= 0) {
        return arr;
    }
    return arr.reduce((pre, cur) => {
        if(Array.isArray(arr)) {
           return pre.concat(customFlat(cur, depth - 1));
        }
        return pre.concat(cur);
    }, []);
}
```

**Array.prototype.flatMap**

The `flatMap()` method first maps each element of the array (with a depth value of 1) using the mapping function, and then compresses the result into a new array.

Return value: a new array with each element being the result of the callback function.

**Syntax**

```
arr.flatMap(function callback(currentVal[, index[, array]]) {

}[, thisArg])
```

- callback: a function called to generate a new array

- currentVal: The element currently being processed by the array

- index: optional, the index of the element being processed

- array: optional, the array to be called

- thisArg: this value used when executing the callback function

Example:

```
let arr = ['My name', 'is', '', 'Lisa'];
let newArr1 = arr.flatMap(cur => cur.split(' '));
let newArr2 = arr.map(cur => cur.split(' '));
console.log(newArr1); // ["My", "name", "is", "", "Lisa"]
console.log(newArr2); // [["My", "name"], ["is"], [""], ["Lisa"]]
```

### Object.fromEntries

The `fromEntries()` method converts a list of key-value pairs into an object

Return value: a new object

Syntax

```
Object.fromEntries(iterable)
```

iterable: Array, Map and other iterable objects

Example:

```
let map = new Map([['a', 1], ['b', 2]]);
let mapToObj = Object.fromEntries(map);
console.log(mapToObj);  // {a: 1, b: 2}

let arr = [['a', 1], ['b', 2]];
let arrToObj = Object.fromEntries(arr);
console.log(arrToObj);   // {a: 1, b: 2}

let obj = {a: 1, b: 2};
let newObj = Object.fromEntries(
  Object.entries(obj).map(
    ([key, val]) => [key, val * 2]
  )
);
console.log(newObj);   // {a: 2, b: 4}
```

### String.prototype.{trimStart, trimEnd}

**String.prototype.trimStart**

The `trimStart()` method is used to remove whitespace characters at the beginning of a string.

`trimLeft()` is its alias.

Return value: a new string with the spaces on the left side of this string removed.

Syntax

```
str.trimStart();
str.trimLeft();
```

Example:

```
let str = '    a b cd  ';
str.trimStart();   // 'a b cd  '
str.trimLeft();    // 'a b cd  '
```

**String.prototype.trimEnd**

The `trimEnd()` method is used to remove whitespace characters at the end of a string.

Return value: a new string, the spaces on the right side of this string have been removed

Syntax

```
str.trimEnd()
str.trimRight()
```

Example:

```
let str = '    a b cd  ';
str.trimEnd();                           // '    a b cd'
str.trimRight();                         // '    a b cd'
```

### Symbol.prototype.description

description is a read-only property

Return value: It returns a string with an optional description of the Symbol object

Syntax

```
Symbol('myDescription').description;
Symbol.iterator.description;
Symbol.for('foo').description;
```

Example:

```
Symbol('foo').description;      // 'foo'
Symbol().description;           // undefined
Symbol.for('foo').description;  // 'foo'
```

### Optional catch binding

Optional capture binding, allowing omission of the catch binding and the parentheses following it

Previous usage:

```
try {

} catch(err) {
  console.log('err', err);
}
```

ES10 usage:

```
try {

} catch {

}
```

### The strengthening power of JSON.stringify()

**JSON.stringify()** In ES10, the problem of display errors for some out-of-range Unicode has been fixed. Therefore, characters within 0xD800-0xDFF will cause display errors because they cannot be encoded into UTF-8. In ES10, it will use escape characters to process these characters instead of encoding, so they will be displayed normally.

```
JSON.stringify('😊'); // '"😊"'
```

### Revision Function.prototype.toString()

The previous toString method from `Object.prototype.toString()` is now a `Function.prototype.toString()` method that returns a string representing the source code of the current function. Previously, only this function would be returned, without spaces, comments, etc.

```
function foo() {
    // es10
    console.log('imooc')
}
console.log(foo.toString());
// function foo() {
//     // es10
//     console.log('imooc')
// }
```

## ES11

ES2020 (ES11) has added the following new features

- Nullish coalescing Operator

- Optional chaining

- globalThis

- BigInt

- String.prototype.matchAll()

- Promise.allSettled()

- Dynamic import (import on demand)

### Nullish coalescing Operator

**Null value merging operator ( ?? )**

The null value merging operator ( `??` ) is a logical operator that returns the right operator when the left operand is null or undefined , otherwise returns the left-hand operator.

```
undefined ?? 'foo'  // 'foo'
null ?? 'foo'  // 'foo'
'foo' ?? 'bar' // 'foo'
```

**Logical OR operator ( || )**

The logical OR operator ( || ) will return the right operand when the left operand is false, that is, if you use || to set default values ​​for some variables , unexpected situations may occur. For example, 0, '', NaN, false:

```
0 || 1  // 1
0 ?? 1  // 0

'' || 'bar'  // 'bar'
'' ?? 'bar'  // ''

NaN || 1  // 1
NaN ?? 1  // NaN

false || 'bar'  // 'bar'
false ?? 'bar'  // false
```

Note

You cannot use ?? with AND ( && ) OR ( || ), and an error will be reported.

```
null || undefined ?? "foo"; // throw SyntaxError
true || undefined ?? "foo"; // throw SyntaxError
```

### Optional chaining Optional chaining

The optional chain operator ( ?. ) allows reading the value of a property located deep in the chain of connected objects without having to explicitly verify that each reference in the chain is valid. The ?. operator functions similarly to the . chaining operator, except that when referenced as null or undefined An error will be reported, and the return value of the link expression is undefined .

Previous writing:

```
const street = user && user.address && user.address.street;
const num = user && user.address && user.address.getNum && user.address.getNum();
console.log(street, num);
```

ES11:

```
const street2 = user?.address?.street;
const num2 = user?.address?.getNum?.();
console.log(street2, num2);
```

Note: Optional chains cannot be used for assignment:

```
let object = {};
object?.property = 1; // Uncaught SyntaxError: Invalid left-hand side in assignment
```

### globalThis

Previously, in the Web, global objects could be obtained through window and self , but in node.js, global must be used.

In loose mode, you can return this in a function to get the global object, but in strict mode and module environment, this will return undefined .

In the past, to obtain the global object, you could define a function:

```
const getGlobal = () => {
    if (typeof self !== 'undefined') {
        return self
    }
    if (typeof window !== 'undefined') {
        return window
    }
    if (typeof global !== 'undefined') {
        return global
    }
    throw new Error('can't find global object')
}

const globals = getGlobal()
console.log(globals)
```

globalThis now provides a standard way to obtain the global object's own value in different environments.

### BigInt

BigInt is a built-in object used to create integers larger than 2⁵³ — 1 (the largest number that can be created by Number). Can be used to represent arbitrarily large integers

**How to define a BigInt**

- Add n after an integer literal, for example 10n

- Call function BigInt() and pass an integer or string value, such as BigInt(10)

**Features of BigInt**

- BigInt cannot be used in methods in Math objects;

- BigInt cannot be mixed with any Number instance, both must be converted to the same type. Note, however, that BigInt may lose precision when converted to Number.

- When using BigInt, operations with decimals are rounded down

- BigInt and Number are not strictly equal, but are loosely equal

```
0n === 0 // false
0n == 0  // true
```

- BigInt and Number can be compared

```
2n > 2   // false
2n > 1   // true
```

- BigInt and Number can be mixed in an array for sorting

```
const mixed = [4n, 6, -12n, 10, 4, 0, 0n];
mixed.sort();  // [-12n, 0, 0n, 10, 4n, 4, 6]
```

- BigInt wrapped by Object is compared using the comparison rules of object, and is only equal when comparing with the same object.

```
0n === Object(0n); // false
Object(0n) === Object(0n); // false
const o = Object(0n);
o === o // true
```

**BigInt methods**

- BigInt.asIntN() — Converts a BigInt value to a signed integer between -2^(width-1) and 2^(width-1) — 1.

- BigInt.asUintN() — Converts a BigInt value to an unsigned integer between 0 and 2^(width) — 1.

- BigInt.prototype.toLocaleString() — Returns the language-sensitive form of this number as a string. Override the Object.prototype.toLocaleString() method.

- BigInt.prototype.toString() — Returns a string representing the specified number in the specified base. Override the Object.prototype.toString() method.

- BigInt.prototype.valueOf() — Returns the primitive value of the specified object. Override the Object.prototype.valueOf() method.

**Why is there a Bigint proposal?**

In JavaScript, Number.MAX_SAFE_INTEGER represents the maximum safe number, and the calculated result is 9007199254740991, that is, there will be no loss of precision within this number range (except for decimals). But once it exceeds this range, js will have inaccurate calculations, which has to rely on some third-party libraries to solve large number calculations. Therefore, the official proposed BigInt to solve this problem.

### String.prototype.matchAll()

Returns an iterator containing all results matching the regular expression and grouped capturing groups.

```
const regexp = /t(e)(st(\d?))/g;
const str = 'test1test2';

const array = [...str.matchAll(regexp)];
console.log(array[0]);  // ["test1", "e", "st1", "1"]
console.log(array[1]); // ["test2", "e", "st2", "2"]
```

### Promise.allSettled()

Class method that returns a promise after all given promises have been fulfilled or rejected, with an array of objects, each object representing the corresponding promise result.

```
Promise.allSettled([
  Promise.resolve(33),
  new Promise((resolve) => setTimeout(() => resolve(66), 0)),
  99,
  Promise.reject(new Error("an error")),
]).then((values) => console.log(values)); 

// [
//   { status: 'fulfilled', value: 33 },
//   { status: 'fulfilled', value: 66 },
//   { status: 'fulfilled', value: 99 },
//   { status: 'rejected', reason: Error: an error }
// ]
```

### Dynamic import (import on demand)

import You can load a module when needed.

```
button.addEventListener('click', event => {
  import('./dialogBox.js')
  .then(dialogBox => {
    dialogBox.open();
  })
  .catch(error => {
    /* Error handling */
  })
});
```

## ES12

ES 2021 (ES12) has added the following new features

- Logical operators and assignment expressions (&&=, ||=, ??=)

- String.prototype.replaceAll()

- Number separator

- Promise.any

### Logical operators and assignment expressions (&&=, ||=, ??=)
1. &&=

The logical AND assignment operator x &&= y is equivalent to x && (x=y) : meaning that when x is true, x = y.

```
let a = 1;
let b = 0;

a &&= 2;
console.log(a); // 2

b &&= 2;
console.log(b);  // 0
```

2. ||=

The logical OR assignment operator x ||= y is equivalent to x || (x = y) : meaning x = y only if x is false.

```
const a = { duration: 50, title: '' };

a.duration ||= 10;
console.log(a.duration);  // 50

a.title ||= 'title is empty.';
console.log(a.title);  // "title is empty"
```

3. ??=

The logical null assignment operator x ??= y is equivalent to x ?? (x = y) : meaning that x = y only if x is null or undefined.

```
const a = { duration: 50 };

a.duration ??= 10;
console.log(a.duration);  // 50

a.speed ??= 25;
console.log(a.speed);  // 25
```

### String.prototype.replaceAll()

Returns a new string in which all parts of the string that match pattern will be replaced by replacement. The original string remains unchanged.

- pattern can be a string or RegExp;

- replacement can be a string or a function that is called each time it is matched.

```
'aabbcc'.replaceAll('b', '.');  // 'aa..cc'
```

When searching for a value using a regular expression, it must be global:

```
'aabbcc'.replaceAll(/b/, '.');  // TypeError: replaceAll must be called with a global RegExp

'aabbcc'.replaceAll(/b/g, '.');  // "aa..cc"
```

### Number separator

ES12 allows JavaScript values ​​to use underscores (_) as separators, but does not specify the number of digits to separate them:

```
123_00
```

Decimal and scientific notation can also use delimiters:

```
0.1_23
1e10_00
```

NOTE:

- Cannot be placed at the front or last of the value;

- Two or more delimiters cannot be connected together;

- There cannot be separators before and after the decimal point;

- In scientific notation, there cannot be separators before or after e or E.

### Promise.any
The method accepts a set of Promise instances as parameters, wraps them into a new Promise instance and returns it.

As long as one parameter instance becomes fulfilled, the wrapper instance will become fulfilled; if all parameter instances become rejected, the wrapper instance will become rejected.

```
const promise1 = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("promise1");
      //  reject("error promise1 ");
    }, 3000);
  });
};
const promise2 = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("promise2");
      // reject("error promise2 ");
    }, 1000);
  });
};
const promise3 = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve("promise3");
      // reject("error promise3 ");
    }, 2000);
  });
};
Promise.any([promise1(), promise2(), promise3()])
  .then((first) => {
    // As long as one request is successful, the first successful request is returned
    console.log(first); // return promise2
  })
  .catch((error) => {
    // All three requests fail to come here
    console.log("error", error);
  });
```
