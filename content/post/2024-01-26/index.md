---
title: 'Analysis of ES basic knowledge points'
date: 2024-01-26T16:22:38+08:00
draft: true
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/0*OZL33xhjFl0XFUmv"
categories: ["Javascript"]
tags: ["Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "JavaScript is an implementation of the ECMAScript specification. This section focuses on sorting out commonly tested knowledge points in ECMAScript..."
---

JavaScript is an implementation of the ECMAScript specification. This section focuses on sorting out commonly tested knowledge points in ECMAScript, and then analyzing some easily arising questions.

# Knowledge Points

- Variable Types

  - Classification and judgment of JS data types

  - Value types and reference types

- Prototype and prototype chain (inheritance)

  - Definition of prototype and prototype chain

  - Inheritance Syntax

- Scope and Closure

  - Execution Context

  - this

  - What is a closure?

- Asynchronous

  - Synchronous vs Asynchronous

  - synchrony and single-threaded

  - Front-end asynchronous scenarios

- Examination of ES6/7 new standards

  - Arrow functions

  - Module

  - Class

  - Set and Map

  - Promise

---

## Variable Types

JavaScript is a weakly-typed scripting language, that is, it doesn’t require specifying the type when you define a variable; the type will be automatically determined during program execution.

### ECMAScript defines six primitive types:

- Boolean

- String

- Number

- Null

- Undefined

- Symbol (newly defined in ES6)

Note: The primitive types do not include Object.

> Topic: What methods are used for type judgment?

**typeof**

The values obtained by `typeof xxx` are of the following types: `undefined`, `boolean`, `number`, `string`, `object`, `function`, `symbol`. It’s pretty straightforward, so no need for individual demonstrations. There are three points to note here:

- The result of `typeof null` is o`bject; actually, this is a bug of typeof, null is primitive value, not a reference type.

- The result `of typeof [1, 2]` is `object`, there is no `array` in the result. Reference types are all objects except functions.

- The value obtained by using `typeof Symbol()` for the `symbol` type is `symbol`, which is a new knowledge point added in ES6.

**instanceof**

It is used for correspondence between instances and constructors. For example, to determine if a variable is an array, `typeof` cannot be used, but you can use `[1, 2] instanceof Array` to judge. Because `[1, 2]` is an `array`, its constructor is `Array`. Similarly:

```
function Foo(name) {
 this.name = name
}
var foo = new Foo('bar')
console.log(foo instanceof Foo) // true
```

> Topic: Difference between value types and reference types

### Value type vs Reference type

In addition to primitive types, ES also has reference types. Among the types identified by `typeof` mentioned above, only `object` and `function` are reference types, others are value types.

Based on the way variable types are passed in JavaScript, it is divided into `value` types and `reference` types. Value types include `Boolean`, `String`, `Number`, `Undefined`, `Null`, and `reference` types include all classes of the `Object` category, such as `Date`, `Array`, `Function`, etc. In terms of parameter passing, value types are passed by value and reference types are passed by sharing.

Let’s look at the main differences between the two and what needs to be noticed in practical development through a small question below.

```
// value types
var a = 10
var b = a b = 20
console.log(a) // 10
console.log(b) // 20
```

In the above code, both a and b are value types. They are assigned and modified separately, and there is no influence on each other. Now let’s look at an example of reference type:

```
// reference types
var a = {x: 10, y: 20}
var b = a b.x = 100
b.y = 200
console.log(a) // {x: 100, y: 200}
console.log(b) // {x: 100, y: 200}
```

In the above code, both `a` and `b` are reference types. After executing `b = a`, when the property value of `b` is modified, the value of `a` also changes. This is because both a and `b` are reference types, pointing to the same memory address, i.e., they both reference the same value. Therefore, when `b` modifies the property, the value of `a` changes accordingly.

Let’s further explain this with the help of the question below.

**State the execution result of the following code and analyze the reason.**

```
function foo(a){
 a = a * 10; }
function bar(b){
 b.value = 'new'; }
var a = 1;
var b = {value: 'old'};
foo(a);
bar(b);
console.log(a); // 1
console.log(b); // value: new
```

Through the execution of the code, you will find that:

- The value of a does not change

- But the value of b has changed

This is because the `Number` type `a` is passed by value, while the `Object` type `b` is passed by sharing.

The reason for this design in JS is: for types that are passed by value, `a` copy is stored in stack memory. This type of type generally does not occupy too much memory, and passing by value ensures its access speed. The types that are passed by sharing are copying its reference, not copying its value (pointer in C language), which ensures that oversized objects and others will not cause memory waste due to constant copying of content.

Reference types are often used in code in the following way, or they are prone to unnoticed errors!

```
var obj = {
 a: 1,
 b: [1,2,3] }
var a = obj.a
var b = obj.b a = 2 b.push(4)
console.log(obj, a, b)
```

Although obj itself is a variable of reference type (object), both `a` and `b` inside it are value types and reference types respectively. Assigning `a` value to a will not change `obj.a`, but operations on `b` will be reflected on the obj object.

## Prototype and prototype chain

JavaScript is a language based on prototypes, understanding prototypes is quite simple, but extremely important. Let’s understand the concept of prototypes in JavaScript through questions.

> Topic: How to understand the prototypes in JavaScript

To answer this question, you can remember and understand several key points:

- All reference types (`arrays`, `objects`, `functions`) have object characteristics, that is, they can freely extend properties (except `null`).

- All reference types (`arrays`, `objects`, `functions`) have a `__proto__` property, the value of which is a normal object.

- All functions have a prototype property, the value of which is also a normal object.

- All reference types (arrays, objects, functions), the value of the `__proto__` property points to the prototype property value of its constructor.

Let’s explain this with code, and you can run the following code to see the results:

```
// Key point one: Freely extend properties
var obj = {}; obj.a = 100;
var arr = []; arr.a = 100;
function fn () {}
fn.a = 100;
// Key point two: __proto__
console.log(obj.__proto__);
console.log(arr.__proto__);
console.log(fn.__proto__);
// Key point three: Functions have prototypes
console.log(fn.prototype)
// Key point four: The value of the __proto__ property of a reference type points to the prototype property value of its constructor
console.log(obj.__proto__ === Object.prototype)
```

### Prototypes

Let’s start with a simple example code.

```
// Constructor
function Foo(name, age) {
 this.name = name
}
Foo.prototype.alertName = function () {
 alert(this.name) }
// Creating an instance
var f = new Foo('zhangsan') f.printName = function () {
 console.log(this.name) }
// Testing
f.printName()
f.alertName()
```

Executing `printName` is easy to understand, but what happened when executing `alertName`? Remember another key point here. When trying to get a certain property of an object, if the object itself does not have this property, it will go to its `__proto__` (that is, its constructor’s prototype) to find it, so `f.alertName` will find `Foo.prototype.alertName`.

So how do you determine whether this property is the object’s own property? Use `hasOwnProperty`, a common place is when traversing an object.

```
var item
for (item in f) {
 // Use hasOwnProperty, a common place is when traversing an object.
 // Advanced browsers have already shielded p roperties from prototypes in for in, 
 // but it is suggested here that you should add this judgment to ensure the robustness of the program.
 if (f.hasOwnProperty(item)) {
 console.log(item)
 }
}
```

> Topic: How to understand the prototype chain in JS

### Prototype Chain
Continuing with the above example, what happens when f.toString() is executed?

```
...
// Testing
f.printName()
f.alertName()
f.toString()
```

Because f itself does not have toString(), and there is no toString in ``f.__proto__``(that is, `Foo.prototype`). Recall the sentence just now — when trying to get a certain property of an object, if the object itself doesn’t have this property, it will look for it in its `__proto__` (that is, its constructor’s prototype).

If toString is not found in `f.__proto__`, continue to look for it in `f.__proto__.__proto__`, because `f.__proto__` is just an ordinary object!

`f.__proto__` is `Foo.prototype`, which does not find toString, keep looking up.
`f.__proto__.__proto__` is `Foo.prototype.__proto__`. As `Foo.prototype` is just an ordinary object, so `Foo.prototype.__proto__` is `Object.prototype`, where toString can be found.
Therefore, f.toString ultimately corresponds to `Object.prototype.toString`.
If you keep looking up like this, you’ll find it’s a chain-like structure, so it’s called a “prototype chain”. If it cannot be found all the way to the top level, it declares failure and returns undefined. What is the top level — `Object.prototype.proto === null`.

**this in the Prototype Chain**

All the methods obtained and executed from the prototype or higher levels of prototypes, the `this` within them when executing, points to the object that triggers the event. Thus, the this in both `printName` and `alertName` is `f`.

## Scope and Closure

Scope and closure are the most likely knowledge points to be tested in front-end interviews. For example, the following question:

> Topic: Now there is a piece of HTML code. The requirement is to write code. When you click on a link with a certain number, an alert pops up with its number.

```
<ul>
 <li>number 1，Click me please pop up 1</li>
 <li>2</li>
 <li>3</li>
 <li>4</li>
 <li>5</li>
</ul>
```

If you don’t know that this question involves closure, you might write the following code:

```
var list = document.getElementsByTagName('li');
for (var i = 0; i < list.length; i++) {
 list[i].addEventListener('click', function(){
 alert(i + 1)
 }, true) }
```

In fact, you will find that it always pops up 6 when executed. At this point, you should solve it through closures:

```
var list = document.getElementsByTagName('li');
for (var i = 0; i < list.length; i++) {
 list[i].addEventListener('click', function(i){
 return function(){
 alert(i + 1)
 }
 }(i), true) }
```

To understand closures, we need to start with the “execution context”.

### Execution Context

Let’s talk about a knowledge point about variable hoisting. In the interview, you may encounter the following question, where many candidates give wrong answers:

> Topic: Tell the result of the following execution (the author here directly commented the output)

```
console.log(a) // undefined
var a = 100
fn('zhangsan') // 'zhangsan' 20
function fn(name) {
 age = 20
 console.log(name, age)
 var age
}
console.log(b); // throw error
// Uncaught ReferenceError: b is not defined
b = 100;
```

Before a piece of JS script (i.e., within a `<script>` tag) is executed, it needs to be parsed first (hence, JS is an interpreted script language). During parsing, a **global execution context environment** is created first, taking all the variables and function declarations that are about to be executed in the code (internal function is not included because you don’t know when the function will be executed). Variables are temporarily assigned to `undefined`, and functions are declared to be available. After this step is done, the program is officially executed. Emphasize again, this is the work that starts before the code execution.

Let’s look at the small test question above. Why is `a` `undefined`, but `b` throws an error. In fact, JS will “parse the entire document” before the code is executed. It finds `var a`, knows there is a variable `a`, and stores it in the execution context. But `b` didn’t find the var keyword, so it didn’t “reserve space” in the execution context in advance. So when the code is executed, the early-present `a` has a record, but the value hasn’t been assigned yet, which is `undefined`. While `b` is not found in the execution context and naturally throws an error (`b` reference not found).

In addition, before a function is executed, a **function execution context environment** is also created, which is similar to the **global context**, but the **function execution context** adds `this`, arguments, and function parameters. It’s easy to understand parameters and `arguments`, but we need to specifically explain `this`.

To sum up:

- **Range**: a `<script>` tag, a JS file, or a function

- **Global context**: variable definition, function declaration

- **Function context**: variable definition, function declaration, this, arguments

### this

Firstly, understand a very important concept — the value of `this` can only be confirmed during execution, not during definition! Why? — Because `this` is part of the execution context, and the execution context needs to be determined before the code is executed, not when it is defined. As shown in the following example.

```
var a = {
 name: 'A',
 fn: function () {
 console.log(this.name)
 }
}a.fn() // this === a
a.fn.call({name: 'B'}) // this === {name: 'B'}
var fn1 = a.fn
fn1() // this === window
```

`this` will have different execution results, mainly focused on the following scenarios:

- As a constructor, within the constructor.

- As an object property, as in the above code `a.fn()`.

- As a regular function, as in the above code `fn1()`.

- Used for call, apply, bind, like in the above code `a.fn.call({name: ‘B’})`.

Now let’s explain what is the scope and scope chain, scope chain and scope are also frequently asked questions.

> Topic: How to understand the scope and scope chain in JS

### Scope

Before ES6, JS didn’t have block-level scopes. For example:

```
if (true) {
 var name = 'zhangsan'
}
console.log(name)
```

From the above example, we can understand the concept of scope, which is an independent territory, so that variables will not leak out and be exposed.
The name above has been exposed, so **JS doesn’t have block-level scopes, only global scope and function scope**.

```
var a = 100
function fn() {
 var a = 200
 console.log('fn', a) }
console.log('global', a)
fn()
```

Global scope is the outermost scope. If we write many lines of JS code and the variable definitions are not enclosed within functions, then they are all in the global scope. The drawback of this is that it is very prone to collisions and conflicts.

```
// Code written by a
var data = {a: 100}
// Code written by b
var data = {x: true}
```

This is why all the code in jQuery, Zepto, and other libraries are placed in `(function() {…})()`. Because all the variables inside will not be leaked and exposed, they won’t pollute the outside, and they won’t affect other libraries or JS scripts. This is a manifestation of function scope.

*Note: ES6 started to introduce block-level scope, which can be defined by using let, as follows:*

```
if (true) {
 let name = 'zhangsan'
}
console.log(name) // Error, because the name defined by let is in this block level scope (if).
```

### Scope Chain

First, let’s understand what is called a “free variable”. In the following code, `console.log(a)` needs to get the variable `a`, but `a` is not defined in the current scope (compare with `b`). A variable that is not defined in the current scope is called a “free variable”. How to get `a` “free variable”? — Look for it in the parent scope.

```
var a = 100
function fn() {
 var b = 200
 console.log(a)
 console.log(b) }
fn()
```

What if the parent scope doesn’t have it either? It continues to look up step by step until it reaches the global scope. If it still hasn’t found it, it gives up. This step-by-step relationship is called the “scope chain”.

```
var a = 100
function F1() {
 var b = 200
 function F2() {
 var c = 300
 console.log(a) // Free variable, look for it in the parent scope along the scope chain
 console.log(b) // Free variable, look for it in the parent scope along the scope chain
 console.log(c) // Variable in the current scope
 }
 F2()
}
F1()
```

### Closure

I’ve explained these concepts, let’s look at an example to understand closures.

```
function F1() {
 var a = 100
 return function () {
 console.log(a)
 }
}
var f1 = F1()
var a = 200
f1()
```

Free variables will be located from the scope chain, but they rely on the scope chain at the time of function definition, not the execution of the function, the above example is a closure. Closures mainly have two use cases:

- The function is returned as a value, as in the previous example.

- The function is passed as a parameter, as shown in the example below.

```
function F1() {
 var a = 100
 return function () {
 console.log(a)
 }
}
function F2(f1) {
 var a = 200
 console.log(f1())
}
var f1 = F1()
F2(f1)
```

At this point, looking back at the beginning — “Scope and Closure” which is a part of click and pop up alert code viewing closures, it will be easy to understand.

## Asynchronous

Asynchronous and Synchronous are also commonly tested topics in interviews, the author will explain the difference between synchronous and asynchronous below.

### Synchronous vs Asynchronous

Consider the following demo; according to the program’s expressed intent, it should first print 100, print 200 after a second, and finally print 300. But in actual operation, that’s not the case at all.

```
console.log(100)
setTimeout(function () {
 console.log(200)
}, 1000)
console.log(300)
```

Comparatively, the following program first prints 100, then displays 200 (wait for user confirmation), and finally prints 300. This execution effect meets the expected requirements.

```
console.log(100)
alert(200) //confirm after 1 second
console.log(300)
```

What’s the difference between these two? — The first example’s intermediate steps do not block the subsequent program’s operations at all, while the second example does block the subsequent operations. The former behavior is known as Asynchronous (the latter is called Synchronous), which does not block the subsequent program execution.

### Asynchronicity and Single Threaded

The fundamental reason that JS requires asynchronicity is because JS is single-threaded, which means it can only do one thing at a time and can’t lie in two boats ~ do two things at once.

An Ajax request takes 5 seconds due to slow network speed. If it is synchronous, the page will be stuck here for those 5 seconds and nothing can be done. If it’s asynchronous, it’s much better as the 5 seconds waiting won’t delay anything else. That 5-second waiting period is due to a slow network speed, not because of JS.

Speaking of single-threaded, let’s look at a real exam question:

> Topic: Explain the execution process and result of the following code.

```
var a = true;
setTimeout(function(){
 a = false;
}, 100)
while(a){
 console.log('while execut') }
```

This is a very puzzling question, numerous candidates assumed that 100ms later, since a becomes false, the while loop will stop. However, this is not the case. Because JavaScript is single-threaded, once it enters the while loop, it doesn’t have the ‘time’ (or thread) to run the timer. Thus, running this code results in an infinite loop!

### Front-end asynchronous scenarios

- Timing functions: setTimeout, setInterval

- Network request, like Ajax, `<img>` loading

Ajax code example

```
console.log('start') $.get('./data1.json', function (data1) {
 console.log(data1)
})
console.log('end')
```

img code example (commonly used for dot statistics)

```
console.log('start')
var img = document.createElement('img')
// or img = new Image()
img.onload = function () {
 console.log('loaded')
 img.onload = null
}
img.src = '/xxx.png'
console.log('end')
```

## Examination of ES6/7 new standards

> Topic: What’s the difference between `this` in ES6 arrow functions and in regular functions?

### Arrow functions

Arrow functions are a new way of defining functions in ES6. A function like `function name(arg1, arg2) {…}` can be defined using `(arg1, arg2) => {…}`. Examples are as follows:

```
// JS regular function
var arr = [1, 2, 3]
arr.map(function (item) {
 console.log(index)
 return item + 1
})
// ES6 arrow function
const arr = [1, 2, 3]
arr.map((item, index) => {
 console.log(index)
 return item + 1
})
```

The purpose of arrow functions is twofold. Firstly, it’s more concise to write. Secondly, it can solve the problem that this is a global variable when functions are executed before ES6. See the following code:

```
function fn() {
 console.log('real', this) // {a: 100} ，The real value of 'this' under this scope.
 var arr = [1, 2, 3]
 // JS regular function
 arr.map(function (item) {
 console.log('js', this) // window 。The second argument of the map function is undefined. If 'this' is not set, it is defaulted to the top-level object, which is 'window' in the browser.
 return item + 1
 })
 // ES6 arrow function
 arr.map(item => {
 console.log('es6', this) // {a: 100} 。For the arrow function, the printed value is the 'this' of the parent scope.
 return item + 1
 })
}
fn.call({a: 100})
```

> Topic: How to use ES6 modules?

### Module

The syntax of module in ES6 is simpler, directly see examples.
If you only output a unique object, use export default. The code is as follows：

```
// Create a file 'util1.js', the content is as follows.
export default {
 a: 100
}
// Create a file 'index.js', the content is as follows.
import obj from './util1.js'
console.log(obj)
```

If you want to output multiple objects, you can’t use `default`, and you need to add `{…}` when you `import`. The code is as follows:

```
// Create a file 'util2.js', the content is as follows.
export function fn1() {
 alert('fn1') }
export function fn2() {
 alert('fn2') }
// Create a file 'index.js', the content is as follows.
import { fn1, fn2 } from './util2.js'
fn1()
fn2()
```

> Topic: What’s the difference between ES6 ‘class’ and regular constructor functions?

### Class

class has always been a keyword (reserved word) in JS, but it has not been formally used until ES6. The ‘class’ in ES6 is to replace the previous method of initializing objects with constructor functions, which is more object-oriented in syntax. For example:

JS constructor function syntax:

```
function MathHandle(x, y) {
 this.x = x;
 this.y = y; }
MathHandle.prototype.add = function () {
 return this.x + this.y;
};
var m = new MathHandle(1, 2);
console.log(m.add())
```

ES6 class syntax:

```
class MathHandle {
 constructor(x, y) {
 this.x = x;
 this.y = y;
 }
 add() {
 return this.x + this.y;
 }
}
const m = new MathHandle(1, 2);
console.log(m.add())
```
Consider the following points, all about class syntax:

- class is a new syntax form, it’s `class Name {…}`, which is completely different from function syntax

- Comparing the two, the content of the constructor function body should be placed in the `constructor` function in the class, `constructor` is the constructor, it is executed by default when initializing the instance.

- The way to write functions in class is `add() {…}`, without the function keyword.

Using class to implement inheritance is much simpler, at least simpler than implementing inheritance with constructor functions. See the following examples:

JS constructor function implements inheritance

```
// animal
function Animal() {
 this.eat = function () {
 console.log('animal eat')
 }
}
// dog
function Dog() {
 this.bark = function () {
 console.log('dog bark')
 }
}
Dog.prototype = new Animal()
// husky
var husky = new Dog()
```

The way class is used in ES6 to implement inheritance

```
class Animal {
 constructor(name) {
 this.name = name
 }
 eat() {
 console.log(`${this.name} eat`)
 }
}
class Dog extends Animal {
 constructor(name) {
 super(name)
 this.name = name
 }
 say() {
 console.log(`${this.name} say`)
 }
}
const dog = new Dog('husky')
dog.say()
dog.eat()
```

Notice the following two points:

- Using `extends` can implement inheritance, which is more in line with the syntax of classical object-oriented languages, like Java.

- The `constructor` of the subclass must execute `super()` to call the `constructor` of the superclass.

> Topic: What new data types have been added in ES6?

### Set and Map

Set and Map are new data structures added in ES6. They extend the current JS array and object, the two important data structures. As they are newly added data structures, they are not widely used currently, but as a front-end programmer, it is necessary to understand them in advance. Let’s summarize the key points of both:

- Set is similar to an array, but arrays allow elements to be duplicated, whereas Set does not allow duplicate elements.

- Map is similar to an object, but regular object’s keys must be strings or numbers, whereas keys of a Map can be of any data type.

#### Set

A Set instance does not allow duplicate elements, which can be demonstrated in the following example. A Set instance can be initialized from an array or by adding elements using add. Duplicate elements will be ignored.

```
// example 1
const set = new Set([1, 2, 3, 4, 4]);
console.log(set) // Set(4) {1, 2, 3, 4}
// example 2
const set = new Set();
[2, 3, 5, 4, 5, 8, 8].forEach(item => set.add(item));
for (let item of set) {
 console.log(item);
}
// 2 3 5 4 8
```

Attributes and methods of a Set instance include:

- **size**: Get the number of elements.

- **add(value)**: Adds an element and returns the Set instance itself.

- **delete(value)**: Deletes an element and returns a boolean indicating whether the deletion was successful.

- **has(value)**: Returns a boolean indicating whether the value is an element of the Set instance.

- **clear()**: Clears all elements without a return value.

```
const s = new Set();
s.add(1).add(2).add(2); // Add element
s.size // 2
s.has(1) // true
s.has(2) // true
s.has(3) // false
s.delete(2);
s.has(2) // false
s.clear();
console.log(s); // Set(0) {}
```

There are various methods to iterate over a Set instance:

- **keys()**: Returns an iterator of key names.

- **alues()**: Returns an iterator of key values. However, as Set structures do not have key names, only key values (or say key names and key values are the same), keys() and values() result in the same values.

- **entries()**: Returns an iterator of key-value pairs.

- **forEach()**: Iterates over each member using a callback function.

```
let set = new Set(['aaa', 'bbb', 'ccc']);
for (let item of set.keys()) {
 console.log(item);
}
// aaa
// bbb
// ccc
for (let item of set.values()) {
 console.log(item);
}
// aaa
// bbb
// ccc
for (let item of set.entries()) {
 console.log(item);
}
// ["aaa", "aaa"]
// ["bbb", "bbb"]
// ["ccc", "ccc"]
set.forEach((value, key) => console.log(key + ' : ' + value))
// aaa : aaa
// bbb : bbb
// ccc : ccc
```

#### Map

The usage of Map is basically the same as an ordinary object. Let’s first look at its feature of using non-string or non-number as a key.

```
const map = new Map();
const obj = {p: 'Hello World'};
map.set(obj, 'OK')
map.get(obj) // "OK"
map.has(obj) // true
map.delete(obj) // true
map.has(obj) // false
```

You need to use `new Map()` to initialize an instance. In the following code, set, get, has, and delete are self-explanatory (this will also be demonstrated later). map.set(obj, ‘OK’) here is using object as the key (not only can it be an object, but any data type is allowed), and map.get(obj) correctly retrieves it later.

The attributes and methods of a Map instance are as follows:

- **size**：Gets the number of members.

- **set**：Sets member’s key and value.

- **get**：Gets the member’s attribute value.

- **has**：Checks whether the member exists.

- **delete**：Deletes a member.

- **clear**：Clears all.

```
const map = new Map();
map.set('aaa', 100);
map.set('bbb', 200);
map.size // 2
map.get('aaa') // 100
map.has('aaa') // true
map.delete('aaa')
map.has('aaa') // false
map.clear()
```

The methods to iterate over a Map instance are:

- **keys()**: Returns an iterator of key names.

- **values()**: Returns an iterator of key values.

- **entries()**: Returns an iterator of all members.

- **forEach()**: Iterates over all members of the Map.

```
const map = new Map();
map.set('aaa', 100);
map.set('bbb', 200);
for (let key of map.keys()) {
 console.log(key);
}
// "aaa"
// "bbb"
for (let value of map.values()) {
 console.log(value);
}
// 100
// 200
for (let item of map.entries()) {
 console.log(item[0], item[1]);
}
// aaa 100
// bbb 200
// or
for (let [key, value] of map.entries()) {
 console.log(key, value);
}
// aaa 100
// bbb 200
```

### Promise

Promise is a specification proposed by CommonJS. It has multiple versions and it has been included in the ES6 standard. ES6 natively supports Promise objects, and libraries like Bluebird and Q can be used for support in non-ES6 environments.

Promise can make callback calls appear as chained calls, making the process more clear and the code more elegant.

In short, promise can be summarized as **three statuses, two processes**, and one method. A quick way to remember is “3–2–1”:

Three statuses: **pending**, **fulfilled**, **rejected**

Two processes:

- pending to fulfilled (resolve)

- pending to rejected (reject)

- One method: then

Of course, there are other concepts like catch, **Promise.all/race**, but we will not expand on them here.
