---
title: 'Do you understand precompilation in JavaScript?'
date: 2024-06-05T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*b4SL3OsAIh-lS3WGYZuA9A.jpeg"
categories: ["Code","Javascript"]
tags: ["Code","Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "Precompilation refers to the operations performed by the interpreter on the code before it is executed. There are mainly two operations: variable declaration and function declaration"
---

## Overview

Precompilation refers to the operations performed by the interpreter on the code before it is executed. There are mainly two operations: variable declaration and function declaration. The reason for these two operations is to ensure that undeclared variables can be used in advance during code execution.

Understanding the underlying compilation principles in JavaScript is essential. Through several examples, we will explore and learn about the foundational principles of precompilation in JavaScript. First, we should understand what the concept of precompilation is.

## Example One

```
console.log(a)
var a = 0;
```

Some might say: Would anyone still write code like this? Isn’t this an obvious mistake? The compiler will definitely throw an error…

But the fact is: the compiler does not throw an error, it just outputs undefined. So some may wonder why it does not throw an error when the definition of the variable is after the output. This is because of an operation in JavaScript’s precompilation: **hoisting**.

Hoisting includes two types: **variable declaration** and **function declaration**.

1. Variable declaration: Hoisting moves the declaration to the top, allowing the variable to be declared in advance, i.e., it informs the compiler about a variable named ‘a’ before the code executes.

2. Function declaration: Similarly, function declaration moves the entire function body to the top, letting the compiler know about the existence of the function body.

So, the reason the compiler does not throw an error and instead outputs undefined is that the interpreter hoists the variable declaration. It’s as if a variable ‘a’ is defined on the first line of the code, but it’s just not assigned a value.

## Example Two

Go ahead and provide the example, and we will delve into the details of the operations performed during the precompilation process.

```
function fn(a, b) {
  console.log(a);
  c = 0
  var c;
  a = 3
  b = 2
  function a() {}
  console.log(b);
  function b() {}
  console.log(b);
}
fn(1)
```

Without seeing the specific example of how the function `fn` is defined and called, it's challenging to accurately determine the three values that will be output. Please provide the function definition and the context in which it's called for a precise answer.

The answer is:

```
function:a
2
2
```

Why is the first output not the passed argument 1, and why is the third not `function: b`? This has to do with the operation order in JavaScript’s precompilation.

The reason the first output is not the argument value 1 and the third output is not `function: b` can be attributed to the operation order during JavaScript’s precompilation phase.

In JavaScript, during the precompilation phase of the execution context, variable and function declarations are hoisted to the top of their containing scope. However, the assignments occur in their original order in the code. This means that functions and variables are known (declared) before the code starts to execute, but their values, if assigned one, are not set until those lines of code are reached during execution.

For the first output not being the argument value 1, it’s possible that a variable with the same name is declared (and possibly assigned) before the function call within the same scope, overshadowing the argument.

Regarding the third output not being `function: b`, if there’s a variable declaration with the same name after the function declaration, the variable declaration (due to hoisting) will overshadow the function declaration. However, since only declarations (not assignments) are hoisted, if the variable b is not assigned a value until after the console.log statement, it would result in `undefined` being logged instead of `function: b`.

This intricacy highlights the importance of understanding hoisting and execution order within JavaScript’s compilation and execution phases.

The underlying operations in the precompilation phase

In the function body, the precompilation roughly has four steps:

1. Create an Activation Object (AO) for the function’s execution context.

2. Identify formal parameter and variable declarations, treat the parameter and variable names as properties of AO, with their values being undefined.

3. Unify the actual arguments with the formal parameters.

4. Search within the function body for function declarations, and assign the function name as the property name of AO with the function body being the value.

In the case of global variables, the steps for precompilation are somewhat simpler:

1. Create a Global Object (GO) for the global execution context.

2. Search for variable declarations, with variable names being the property names of GO, and their values being undefined.

3. Search globally for function declarations, with the function names being the property names of GO, and their bodies being the value.

## Example Three

The third example we need to discuss is the order of calling variables during precompilation within a function body.

```
global=100
function fn(){
  console.log(global);
  global = 200
  console.log(global);
  global = 300
}
fn()
var global;
```

The answer is:

```
undefined
200
```

Why is the first output undefined? This is due to the hoisting we talked about earlier. In JavaScript, the function body first checks if there is a variable named ‘global’ within it, and only if it’s not found will it resort to the global variable. If you remove the code on line 6, the first output would be 100 instead of undefined. This is because, without the variable declaration inside the function body, it will search for ‘global’ in the global variables, where ‘global’ has already been assigned the value of 100.

## Summary

Through these examples, I believe all the clever friends have not only understood the basic concept of precompilation in JavaScript but also appreciated how the underlying logical mechanisms can deeply affect the flow of the code. Precompilation, being a core part of the JavaScript execution mechanism, ensures that all declared identifiers can be accessed at runtime, even if they appear before their actual declarations, through variable hoisting and function hoisting. This mechanism reflects the concept of variable hoisting in JavaScript. However, it’s important to note that variables declared with let and const are hoisted logically but will be in a Temporal Dead Zone (TDZ) until they are assigned a value, making them inaccessible before that point.

In the function body, precompilation includes the creation of AO, handling of formal parameters and variable declarations, binding of actual arguments to formal parameters, and processing of function declarations. Global context precompilation involves the creation of GO, handling of variable declarations, and processing of function declarations.

In summary, the precompilation mechanism in JavaScript is not just a mechanism but a key to deeply understanding the code execution process and optimizing the code flow. By mastering the principles of precompilation, we can control the execution flow of code more precisely, enhancing the readability and stability of the program.
