---
title: '10 JavaScript Tricks You Don’t Know'
date: 2024-02-16T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*ESMQv6s2ta9iDO9H3ChVlg.png"
categories: ["Javascript"]
tags: ["Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "As one of the most popular languages, JavaScript’s syntax is flexible and continuously absorbs new features each year...."
---

As one of the most popular languages, JavaScript’s syntax is flexible and continuously absorbs new features each year. Even seasoned professionals occasionally come across some underestimated JavaScript features and tricks. This article will share these tricks for discussion and exploration.

## Using flatMap

Although some JavaScript methods are barely known, they possess the potential to enhance coding efficiency by resolving unique challenges, such as flatMap().

The array method flatMap() is essentially a combination of map() and flat(). The difference is that flatMap can only flatten 1 level, while flat can specify the number of levels to flatten. FlatMap is slightly more efficient than calling these two methods separately.

**Using flat + map**:

```
const arr = [1, 2, [4, 5], 6, 7, [8]];

// Use map to operate on each element and use flat to flatten the result
const result = arr.map(element => Array.isArray(element) ? element : [element]).flat();

console.log(result); // output: [1, 2, 4, 5, 6, 7, 8]
```

**Using flatMap**:

```
const arr = [1, 2, [4, 5], 6, 7, [8]] ;

console.log(arr.flatMap((element) => element)); 
// output :[1, 2, 4, 5, 6, 7, 8]
```

Although flatMap is a method, it still generates an intermediate array (which refers to a temporary array created for garbage collection). FlatMap is highly suitable for use in scenarios requiring flexibility and readability.

## Array Method Order

javascript has dozens of array methods that can be used in combination. They look something like this:

```
const numbers = [9, 3, 6, 4, 8, 1, 2, 5, 7];

// Sort only for odd numbers and raise them to the power of 3
numbers
  .sort((a, b) => a - b)
  .filter((n) => n % 2 !== 0)
  .map((n) => n ** 3);
```

The above code looks good, but there is an issue — sorting is done for the array before filtering. If the filtering is done before the sorting, we can accomplish fewer tasks, thereby optimizing the code.

```
const numbers = [9, 3, 6, 4, 8, 1, 2, 5, 7];

numbers
  .filter((n) => n % 2 !== 0)
  .sort((a, b) => a - b)
  .map((n) => n ** 3);
```

## Make full use of reduce

When writing JavaScript, sometimes we need to provide data in a key-value grouping format. Most developers will use the .forEach() method or map() method, similar to this:

```
fetch("https://jsonplaceholder.typicode.com/todos/")
  .then(res=>res.json())
  .then(todos=>{
    
    // using forEach() or Map
    const todosForUserMap = {};
    todos.forEach(todo=>{
      if (todosForUserMap[todo.userId]){
        todosForUserMap[todo.userId].push(todo);  
      }else{
        todosForUserMap[todo.userId] = [todo];
      }  
    })

    console.log(todosForUserMap)
  })
```

Here, it’s better to use forEach rather than the map method, because the map method returns a new array, and the performance overhead of array creation and value assignment is relatively large, especially when the data volume is large, this will not happen in forEach.

Another quite clean and readable method is using the reduce method from the array:

```
fetch("https://jsonplaceholder.typicode.com/todos/")
  .then(res=>res.json())
  .then(todos=>{
    
    // using reduce
    const todosForUserMap = todos.reduce((accumulator, todo)=>{
      if (accumulator[todo.userId]) accumulator[todo.userId].push(todo);
      if (!accumulator[todo.userId]) accumulator[todo.userId] = [todo];
      return accumulator;
    },{})

    console.log(todosForUserMap)
  })
```

Note how we’re using an empty object {} as the initial value for the reduce operation. This object becomes the new accumulator.

## Make full use of generator

Generators and iterators might be the least frequently used code by JavaScript developers, as their knowledge is confined to coding interviews. (Because there’s a better syntax sugar async/await? 😂)

The generator is a powerful method to control asynchronous programming, generate iterable objects and multiple values. Generators are different from traditional functions. They can start and stop execution many times. That allows them to generate a lot of values ​​and to continue their execution later, which makes them extremely suitable for managing asynchronous operations, constructing iterators, and dealing with endless data streams.

Imagine a scenario where the amount of data from the database/API might be infinite, and you need to transfer them to the front-end, how would you do it?

In this case, the most common solution in react is infinite loading. If it is in node or native JS, how would you implement a function like infinite loading.

```
async function *fetchProducts(){
  while (true){
    const productUrl = "https://fakestoreapi.com/products?limit=2";
    const res = await fetch(productUrl)
    const data = await res.json()
    yield data;
        // Update the user interface here
        // Or save it in a database or elsewhere
        // Use this for side effects
        // Interrupt the process if some conditions match
  }
}

async function main() {
  const itr = fetchProducts();
        // This should be called based on the user interaction
        // Or other tricks, because you don't want it to load indefinitely.
  console.log( await itr.next() );
}

return main()
```

This is where iterators are really useful, instead of streaming a large amount of request data to local storage or some location. This is one of the ways to perform this operation using asynchronous generators, so we can solve the problem of infinite loading in JS.

## Making Good Use of Console

Console is not just about console.log(). In actual production, a well-packaged log library will typically be used. The console object actually has many useful built-in methods, which can help you improve the quality and readability of your debugging output. Mastering them can help you debug and solve problems in your code more easily.

```
// 1. console.time and console.timeEnd
// Measure the time taken to execute a piece of code. Identify performance bottlenecks in the code and optimize them
console.time('Start fetching data');

fetch('https://reqres.in/api/users')
  .then(response => response.json())
  .then(data => {
    console.timeEnd('Time spent fetching data:');
    // ...code
  });
  
// 2. console.dir
// The console.dir method outputs the properties of an object in a layered format. It is convenient to view the structure of the object as well as all its properties and methods
const promise = new Promise((resolve, reject) => resolve('foo'));
console.dir(promise);

// 3. console.count
// Use the console.count method to count the number of times a specific log message is output. This is very useful for tracking the number of times a specific code path is executed and identifying hotspots in the code
const fun = (x) => console.count(x);

fun('Keqing'); // 1
fun('Ganyu'); // 1
fun('Keqing'); // 2

// 4. console.trace
// trace can output stack traces. It is very useful for understanding the execution flow in the code and identifying the source of a specific log message
const foo = () => console.trace();
const bar = () => foo();
bar();

// 5. console.profile profileEnd
// Measure the performance of a block of code. This is very useful for identifying performance bottlenecks and optimizing code to improve speed and efficiency.
console.profile('MyProfile');

// Code you want to measure performance on
for (let i = 0; i < 100000; i++) {
  // ...code
}

console.profileEnd('MyProfile');
```

## Deep Copy with structuredClone()

Previously, if a developer wanted to do a deep copy of an object, they usually had to rely on a third-party library or manually implement a deep copy, or adopt a hack like const cloneObj = JSON.parse(JSON.stringify(obj));. But it had a lot of shortcomings when dealing with more complex objects containing circular references or data types that don't conform to JSON (such as Map and Set, Blob, etc.)

Now, JavaScript has a built-in method called structuredClone(). This method provides a simple and effective way to deep clone objects and is suitable for most modern browsers and Node.js v17 and above.

```
// Pass the original object to this function and it will return a deep copy with different references and object attribute references

const obj = { name: 'Mike', friends: [{ name: 'Sam' }] };
const clonedObj = structuredClone(obj);

console.log(obj.name === clonedObj); // false
console.log(obj.friends === clonedObj.friends); // false
```

Unlike the well-known JSON.parse(JSON.stringify()), structuredClone() allows you to clone circular references, which makes it the simplest method to use deep copy in JavaScript currently.

## Tagged Templates

Tagged Templates is a more advanced form of template strings (backticks) that allows you to parse template literals using functions.

I only learned about this advanced feature when Next.js 14 was released and people were talking about a certain picture🫡. Although this feature was included in ES6 and has been around for 8 years!!! But I bet there are only a handful of people who know and have used this feature.

If you don’t understand how Tagged Templates work, let me briefly explain it with an example:

```
const checkCurrency = function (currency, amount) {
  const symbol = currency[0] === "USD" ? "$" : "¥";
  console.log(currency[0], "--" ,currency[1])// Outputs: USD -- RMB
  return `${symbol}${amount}`;
};

const amount = 200;
const currency = checkCurrency`USD${amount}RMB`;
console.log(currency); // Outputs: $200

// 1. checkCurrency is a function, the first argument currency of the tagged function contains an array of string values
// 2. The array of strings is composed of the strings in the tagged template in`USD${amount}RMB`, which contains the strings USD and RMB
// 3. So currency[0] refers to the first string USD, and currency[1] corresponds to the second string RMB
// 4. The remaining arguments of the checkCurrency function are then inserted directly into the strings according to the respective expressions, such as amount = 200
// 5. Inside the checkCurrency function, it checks if the first item of the argument array is 'USD', if so, it selects the "$" symbol, otherwise it chooses "¥".
// 6. The function internally concatenates symbol and amount to return a new string, with symbol representing the currency symbol and amount referring to the money passed to the function.
// 7. The returned string is assigned to the currency constant, so the log is $200
```

As you can see, the way Tagged Templates work is to pass all the strings in the template string to the first argument of the function as an array. The remaining arguments are then inserted directly into the strings according to the respective expressions. Tagged Templates pass literal strings and the results of expressions to the function, and then the function can operate and return them in a custom way. This allows developers to appropriately escape and verify inputs when building SQL queries, thus preventing SQL injection attacks.

Tagged template strings can be used for many purposes, such as security, i18n, and localization, etc.

## Nullish Coalescing Operator??

The Nullish Coalescing Operator `??` is a logical operator that returns its right-hand operand when its left-hand operand is null or undefined, otherwise, it returns its left-hand operand.

```
const foo = null ?? 'default string';
console.log(foo);  //output: "default string"

const bar = 0 ?? 'default string'
console.log(bar);  //output: 0
```

What’s worth mentioning about this? Is not `||` enough? Because one question that may confuse many people when learning JS is the difference between false and falsy, and the main difference between `??` and `||` lies in

`??` operator only returns its right-hand operand if the left-hand operand is null or undefined.
`||` operator treats all falsy values ​​of its left operand’s result as its right operand.
Here are some examples:

```
// 1. Using 0 as input 
const a = 0;
console.log(`a || 10 = ${a || 10}`); // a || 10 = 10
console.log(`a ?? 10 = ${a ?? 10}`); // a ?? 10 = 0

// 2. Using an empty string '' as input
const a = ''
console.log(`a || "ABC" = ${a || "ABC"}`); // a || "ABC" = ABC
console.log(`a ?? "ABC" = ${a ?? "ABC"}`); // a ?? "ABC" = 

// 3. Using null as input
const a = null;
console.log(`a || 10 = ${a || 10}`); // a || 10 = 10v
console.log(`a ?? 10 = ${a ?? 10}`); // a ?? 10 = 10

// 4. Using undefined as input
const a = {name: ''}

console.log(`a.name ?? 'varun 1' = ${a.name ?? 'varun 1'}`); 
console.log(`a.name || 'varun 2' = ${a.name || 'varun 2'}`);
// a.name ?? 'varun 1' = 
// a.name || 'varun 2' = varun 2

// 5. Using false as input
const a = false;
console.log(`a || 10 = ${a || 10}`); // a || 10 = 10
console.log(`a ?? 10 = ${a ?? 10}`); // a ?? 10 = false
```

For JS’s falsy value judgement, you can refer to this table: [JavaScript-Equality-Table/](https://dorey.github.io/JavaScript-Equality-Table/).

## Use Symbols as Keys in WeakMap

WeakMap is very similar to Map, but the difference is that its keys can only be objects or symbols, which are stored as weak references.

Why? Because the keys of a WeakMap must be garbage collectable. Most primitive data types can be freely created and have no lifecycle, so they can’t be used as keys. Objects and non-registered symbols can be used as keys because they can be garbage collected — MDN- WeakMap.

This feature means that if there are no other references to the object in memory except for the key, JavaScript engine can perform garbage collection on the object when needed.

```
// Map
let user = { name: "User" };

let map = new Map();
map.set(user, "Ketching");

user = null; // Overwrite the reference by setting to null, 'user' is stored internally in the map, which can be accessed through map.keys()

// WeakMap
let user = { name: "User" };

let weakMap = new WeakMap();
weakMap.set(user, "Ganyu");

user = null; // With WeakMap, 'user' has been deleted from memory 
```

So, what is the use of a WeakMap? Given its characteristics, it can be inferred that WeakMap can be used for custom caching and detecting memory leaks.

By using objects as keys, you can associate cached values with specific objects. When the object is garbage collected, the corresponding WeakMap entry will be automatically deleted, immediately clearing the cache.

In ES14, it has become possible to use symbols as keys in WeakMap, which can make the role of key-value pairs in WeakMap more clear. Because the only primitive type that can be used as keys in WeakMap is the symbol, the symbol guarantees that the key is unique and cannot be recreated.

```
let mySymbol = Symbol('mySymbol');

let myWeakMap = new WeakMap();

let obj = {
    name: 'Frontend Kitten Writer'
};

myWeakMap.set(mySymbol, obj);

console.log(myWeakMap.get(mySymbol)); // Output: {name: 'Frontend Kitten Writer'}
```

## Functional Programming

Since 2015, JavaScript versions have been updated, and this year (2023 ES14) is no exception.

The biggest update in ES14 is that many extra array methods have been added, or complementary methods that don’t cause mutation have been added to the existing array methods. That means they’ll create new arrays based on the original ones rather than directly modifying the original arrays.

The new complementary methods are:

- Array.sort() -> Array.toSorted()

- Array.splice() -> Array.toSpliced()

- Array.reverse() -> Array.toReversed()

The new array methods are:

- Array with()

- Array.findLast()

- Array.findLastIndex()

The key theme of this year is Simpler Functional Programming (fp) and Immutability.

```
// For example, with Array.with(), you had to modify a value of an array with arr[2] = 3;
// This causes a mutation. This is impure! Angry💢 But with the new array methods that don't cause mutations, you can write it like this:
const arr = [5, 4, 7, 2, 1];
const replaced = arr.with(2, 3);

console.log(replaced);  // [5, 4, 3, 2, 1]
```

