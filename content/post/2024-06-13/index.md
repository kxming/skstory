---
title: 'The JSON.stringify() You Don’t Know About'
date: 2024-06-13T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*U5P16wQHFvMFQDjpfDRBjQ.png"
categories: ["Code","Javascript"]
tags: ["Code","Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "We are familiar with JSON.stringify, which is generally used for serialization (deep copy)..."
---

## Concept

We are familiar with `JSON.stringify`, which is generally used for serialization (deep copy). It converts our objects into a JSON string. This method is indeed very convenient in our work, however, this method also has some disadvantages, which we seldom encounter.

## Disadvantages

### 1. Not Friendly to Functions

If an object’s property is a function, this property will be lost during serialization.

```
let obj = {
    name: 'iyongbao',
    foo: function () {
        console.log(`${ this.name }`)
    }
}

console.log(JSON.stringify(obj)); // {"name":"iyongbao"}
```

### 2. Not Friendly to undefined

If an object’s property value is undefined, it will be lost after conversion.

```
let obj = {
    name: undefined
}

console.log(JSON.stringify(obj)); // {}
```

### 3. Not Friendly to Regular Expressions

If an object’s property is a regular expression, it will turn into an empty Object after conversion.

```
let obj = {
    name: 'iyongbao',
    zoo: /^i/ig,
    foo: function () {
        console.log(`${ this.name }`)
    }
}

console.log(JSON.stringify(obj)); // {"name":"iyongbao","zoo":{}}
```

### 4. Array Objects

The above scenarios also occur if it is an array object.

```
let arr = [
    {
        name: undefined
    }
]

console.log(JSON.stringify(arr)); // [{}]
```

## Features

### First

- `JSON.stringify()` will return different results when `undefined`, any `function`, and `symbol`, these three special values, are used as the values of object properties, array elements, or as standalone values.

```
const data = {
  a: "aaa",
  b: undefined,
  c: Symbol("dd"),
  fn: function() {
    return true;
  }
};
JSON.stringify(data); // "{"a":"aaa"}"
```

- When `undefined`, any `function`, and `symbol` are used as array element values, `JSON.stringify()` will serialize them as null.

```
JSON.stringify(["aaa", undefined, function aa() {
    return true
  }, Symbol('dd')])  // "["aaa",null,null,null]"
```

- When `undefined`, any `function`, and `symbol` are serialized by `JSON.stringify()` as standalone values, it will return undefined.

```
JSON.stringify(function a (){console.log('a')})
// undefined
JSON.stringify(undefined)
// undefined
JSON.stringify(Symbol('dd'))
// undefined
```

### Second

The properties of non-array objects cannot be guaranteed to appear in a specific order in the serialized string. As mentioned in the first characteristic, `JSON.stringify()` ignores some special values during serialization, so it cannot guarantee that the serialized string will still appear in a specific order (except for arrays).

```
const data = {
  a: "aaa",
  b: undefined,
  c: Symbol("dd"),
  fn: function() {
    return true;
  },
  d: "ddd"
};
JSON.stringify(data); // "{"a":"aaa","d":"ddd"}"

JSON.stringify(["aaa", undefined, function aa() {
    return true
  }, Symbol('dd'),"eee"])  // "["aaa",null,null,null,"eee"]"
```

### Third

If the value being converted has a `toJSON()` function, the serialization result will be whatever value that function returns, and the values of other properties will be ignored.

```
JSON.stringify({
    say: "hello JSON.stringify",
    toJSON: function() {
      return "today i learn";
    }
  })
// "today i learn"
```

### Fourth

`JSON.stringify()` will serialize Date values normally. In fact, the Date object itself implements the `toJSON()` method (equivalent to `Date.toISOString()`), so the Date object is treated as a string.

```
JSON.stringify({ now: new Date() });
// "{"now":"2024-06-16T12:43:13.577Z"}"
```

### Fifth

Numeric values in the format of `NaN` and `Infinity`, as well as `null`, will all be treated as `null`.

```
JSON.stringify(NaN)
// "null"
JSON.stringify(null)
// "null"
JSON.stringify(Infinity)
// "null"
```

### Sixth

Wrapper objects for booleans, numbers, and strings will be automatically converted to their corresponding primitive values during serialization.

```
JSON.stringify([new Number(1), new String("false"), new Boolean(false)]);
// "[1,"false",false]"
```

### Seventh

Objects of other types, including `Map`/`Set`/`WeakMap`/`WeakSet`, will only serialize enumerable properties.Non-enumerable properties are ignored by default.

```
JSON.stringify( 
    Object.create(
        null, 
        { 
            x: { value: 'json', enumerable: false }, 
            y: { value: 'stringify', enumerable: true } 
        }
    )
);
// "{"y":"stringify"}"
```

### Eighth

We all know that the simplest and most brute-force way to implement deep cloning is serialization: `JSON.parse(JSON.stringify())`. This method of implementing deep cloning will lead to many pitfalls due to the various characteristics of serialization. For example, the circular reference issue we are addressing now.

```
// Executing this method on objects with circular references (objects referencing each other, forming an infinite loop) will throw an error.
const obj = {
  name: "loopObj"
};
const loopObj = {
  obj
};
// Objects form a circular reference, creating a closed loop
obj.loopObj = loopObj;

// Encapsulate a deep clone function
function deepClone(obj) {
  return JSON.parse(JSON.stringify(obj));
}
// Execute deep cloning, throw an error
deepClone(obj)
/**
 VM44:9 Uncaught TypeError: Converting circular structure to JSON
    --> starting at object with constructor 'Object'
    |     property 'loopObj' -> object with constructor 'Object'
    --- property 'obj' closes the circle
    at JSON.stringify (<anonymous>)
    at deepClone (<anonymous>:9:26)
    at <anonymous>:11:13
 */
```

### Ninth

Properties with a `symbol` as the property key will be completely ignored, even if they are explicitly included in the replacer parameter.

```
JSON.stringify({ [Symbol.for("json")]: "stringify" }, function(k, v) {
    if (typeof k === "symbol") {
      return v;
    }
  })

// undefined
```

## Expansion

`JSON.stringify` also has optional second and third parameters.

### 1. Second parameter replacer

The second parameter, `replacer`, can take two forms: a function or an array. As a function, it receives two arguments, the key and value. The function acts similarly to callback functions in array methods like map, filter, etc., executing once for each property value. If `replacer` is an array, the values in the array represent the names of the properties that will be serialized into the JSON string.

#### Used as a function:

```
const data = {
  a: "aaa",
  b: undefined,
  c: Symbol("dd"),
  fn: function() {
    return true;
  }
};
// Without using the replacer parameter
JSON.stringify(data); 
// "{"a":"aaa"}"

// When using the replacer parameter as a function
JSON.stringify(data, (key, value) => {
  switch (true) {
    case typeof value === "undefined":
      return "undefined";
    case typeof value === "symbol":
      return value.toString();
    case typeof value === "function":
      return value.toString();
    default:
      break;
  }
  return value;
})
// "{"a":"aaa","b":"undefined","c":"Symbol(dd)","fn":"function() {\n    return true;\n  }"}"
```

When the replacer function is used, the first argument passed to this function is not the first key-value pair of the object. Instead, an empty string is used as the key, and the value is the entire object’s key-value pairs:

```
const data = {
  a: 2,
  b: 3,
  c: 4,
  d: 5
};
JSON.stringify(data, (key, value) => {
  console.log(value);
  return value;
})
// The first argument passed to the replacer function is {"":{a: 2, b: 3, c: 4, d: 5}}
// {a: 2, b: 3, c: 4, d: 5}   
// 2
// 3
// 4
// 5
```

Implementing a map function

We can also use it to manually implement a similar map function for an object.

```
// Implement a map function
const data = {
  a: 2,
  b: 3,
  c: 4,
  d: 5
};
const objMap = (obj, fn) => {
  if (typeof fn !== "function") {
    throw new TypeError(`${fn} is not a function !`);
  }
  return JSON.parse(JSON.stringify(obj, fn));
};
objMap(data, (key, value) => {
  if (value % 2 === 0) {
    return value / 2;
  }
  return value;
});
// {a: 1, b: 3, c: 2, d: 5}
```

#### Used as an array

When replacer is used as an array, the result is very straightforward. The values in the array represent the names of the properties that will be serialized into the JSON string.

```
const jsonObj = {
  name: "JSON.stringify",
  params: "obj,replacer,space"
};

// Only retain the value of the params property
JSON.stringify(jsonObj, ["params"]);
// "{"params":"obj,replacer,space"}"
```

### 2. Third parameter space

The third parameter, space, is used to control the spacing within the resulting string. Let’s look at an example to understand what this does:

```
const tiedan = {
  name: "Jhon",
  describe: "JSON.stringify()",
  emotion: "like"
};
JSON.stringify(tiedan, null, "--");
// The output is as follows
// "{
// --"name": "Jhon",
// --"describe": "JSON.stringify()",
// --"emotion": "like"
// }"
JSON.stringify(tiedan, null, 2);
// "{
//   "name": "Jhon",
//   "describe": "JSON.stringify()",
//   "emotion": "like"
// }"
```
