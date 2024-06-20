---
title: 'How to optimize complex judgments in JavaScript?'
date: 2024-06-15T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*Umi1OGBvRD25xtjZvKQ4mw.jpeg"
categories: ["Code","Javascript"]
tags: ["Code","Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "When we write JavaScript code, we often encounter situations of complex logical judgments..."
---

## Overview

When we write JavaScript code, we often encounter situations of complex logical judgments. Usually, we can use `if/else` or switch to implement multiple condition judgments. However, there is a problem. As the complexity of the logic increases, the `if/else/switch` in the code will become more and more bloated and harder to understand. So how can we write judgment logic more elegantly?

## Example

```
/**
 * Button click event
 * @param {number} status：1 2 3 4 5
 */
const onButtonClick = (status)=>{
  if(status == 1){
    jumpTo('IndexPage')
  }else if(status == 2){
    jumpTo('FailPage')
  }else if(status == 3){
    jumpTo('FailPage')
  }else if(status == 4){
    jumpTo('SuccessPage')
  }else if(status == 5){
    jumpTo('CancelPage')
  }else {
    jumpTo('Index')
  }
}
```

From the code, we can see the click logic of this button: based on different activity statuses, it does two things, sends log events, and jumps to the corresponding page. It’s easy for everyone to propose a code rewrite scheme using switch:

```
const onButtonClick = (status)=>{
  switch (status){
    case 1:
      console.log('IndexPage')
      break
    case 2:
    case 3:
      jumpTo('FailPage')
      break  
    case 4:
      jumpTo('SuccessPage')
      break
    case 5:
      jumpTo('CancelPage')
      break
    default:
      jumpTo('Index')
      break
  }
}
```

This looks much clearer than using `if/else`. You also discovered a little trick: when the logic for case 2 and case 3 is the same, the execution statement and break can be omitted, then the logic for case 3 will automatically execute for case 2.

Continuous optimization：

```
const actions = {
  '1': ['IndexPage'],
  '2': ['FailPage'],
  '3': ['FailPage'],
  '4': ['SuccessPage'],
  '5': ['CancelPage'],
  'default': ['Index'],
}
const onButtonClick = (status)=>{
  let action = actions[status] || actions['default'],
  jumpTo(action[0])
}
```

The code indeed looks cleaner now. The cleverness of this method lies in: taking the decision conditions as the object’s property names and the processing logic as the object’s property values. When the button is clicked, logical judgments are made by looking up the object properties. This style of writing is especially suited for unary condition judgment.

Is there another way to write this? Use map:

```
const actions = new Map([
  [1, ['IndexPage']],
  [2, ['FailPage']],
  [3, ['FailPage']],
  [4, ['SuccessPage']],
  [5, ['CancelPage']],
  ['default', ['Index']]
])
const onButtonClick = (status)=>{
  let action = actions.get(status) || actions.get('default')
  jumpTo(action[0])
}
```

Writing it this way makes use of the Map object in ES6, doesn’t it feel smoother? What are the differences between a Map object and an Object?

An object usually has its own prototype, so an object always has a prototype key.

The keys of an object can only be strings or Symbols, but the keys of a Map can be any value.

You can easily get the number of key-value pairs in a Map through the size property, while the number of key-value pairs in an object can only be confirmed manually.

To make it more complex, how about adding another layer of judgment?

```
const onButtonClick = (status,identity)=>{
  if(identity == 'guest'){
    if(status == 1){
      //do sth
    }else if(status == 2){
      //do sth
    }else if(status == 3){
      //do sth
    }else if(status == 4){
      //do sth
    }else if(status == 5){
      //do sth
    }else {
      //do sth
    }
  }else if(identity == 'master') {
    if(status == 1){
      //do sth
    }else if(status == 2){
      //do sth
    }else if(status == 3){
      //do sth
    }else if(status == 4){
      //do sth
    }else if(status == 5){
      //do sth
    }else {
      //do sth
    }
  }
}
```

From the example above, we can see that when your logic upgrades to binary judgment, the amount of judgment and your code quantity will double. How can you write it more cleanly at this point?

```
const actions = new Map([
  ['guest_1', ()=>{/*do sth*/}],
  ['guest_2', ()=>{/*do sth*/}],
  ['guest_3', ()=>{/*do sth*/}],
  ['guest_4', ()=>{/*do sth*/}],
  ['guest_5', ()=>{/*do sth*/}],
  ['master_1', ()=>{/*do sth*/}],
  ['master_2', ()=>{/*do sth*/}],
  ['master_3', ()=>{/*do sth*/}],
  ['master_4', ()=>{/*do sth*/}],
  ['master_5', ()=>{/*do sth*/}],
  ['default', ()=>{/*do sth*/}],
])
const onButtonClick = (identity,status)=>{
  let action = actions.get(`${identity}_${status}`) || actions.get('default')
  action.call(this)
}
```

The core logic of the above code is: concatenating the two conditions into a string, and looking up and executing through a Map object with the concatenated condition string as the key and the handling function as the value. This method is particularly useful when making multi-condition judgments.

Of course, implementing the above code using an Object is also similar:

```
const actions = {
  'guest_1':()=>{/*do sth*/},
  'guest_2':()=>{/*do sth*/},
  //....
}

const onButtonClick = (identity,status)=>{
  let action = actions[`${identity}_${status}`] || actions['default']
  action.call(this)
}
```

If concatenating the query conditions into a string feels a bit awkward, then there is another solution, which is to use a Map object with Object objects as the key:

```
const actions = new Map([
  [{identity:'guest',status:1},()=>{/*do sth*/}],
  [{identity:'guest',status:2},()=>{/*do sth*/}],
  //...
])

const onButtonClick = (identity,status)=>{
  let action = [...actions].filter(([key,value])=>(key.identity == identity && key.status == status))
  action.forEach(([key,value])=>value.call(this))
}
```

Here, we can also see the difference between Map and Object; Map can use any type of data as the key.

Let’s increase the difficulty a bit more. Suppose in the case of a guest, the processing logic for status1–4 is all the same, how do we handle it? The worst-case scenario is like this:

```
const actions = ()=>{
  const functionA = ()=>{/*do sth*/}
  const functionB = ()=>{/*do sth*/}
  return new Map([
    [{identity:'guest',status:1},functionA],
    [{identity:'guest',status:2},functionA],
    [{identity:'guest',status:3},functionA],
    [{identity:'guest',status:4},functionA],
    [{identity:'guest',status:5},functionB],
    //...
  ])
}

const onButtonClick = (identity,status)=>{
  let action = [...actions()].filter(([key,value])=>(key.identity == identity && key.status == status))
  action.forEach(([key,value])=>value.call(this))
}
```

Writing it this way can already meet everyday needs. However, to be serious, rewriting functionA four times is still a bit troublesome. If the condition becomes particularly complex, such as having three states for identity and ten states for status, then you would need to define 30 processing logics. Often, many of these logics are the same, which seems unacceptable. In that case, it can be implemented like this:

```
const actions = ()=>{
  const functionA = ()=>{/*do sth*/}
  const functionB = ()=>{/*do sth*/}
  return new Map([
    [/^guest_[1-4]$/,functionA],
    [/^guest_5$/,functionB],
    //...
  ])
}

const onButtonClick = (identity,status)=>{
  let action = [...actions()].filter(([key,value])=>(key.test(`${identity}_${status}`)))
  action.forEach(([key,value])=>value.call(this))
}
```

Here, the advantage of Map becomes even more prominent, as it allows using regex types as keys, which opens up endless possibilities. Suppose the requirement changes so that in every guest scenario, a log event needs to be sent, and different status scenarios also require separate logic processing. In that case, we can write it like this:

```
const actions = ()=>{
  const functionA = ()=>{/*do sth*/}
  const functionB = ()=>{/*do sth*/}
  const functionC = ()=>{/*send log*/}
  return new Map([
    [/^guest_[1-4]$/,functionA],
    [/^guest_5$/,functionB],
    [/^guest_.*$/,functionC],
    //...
  ])
}

const onButtonClick = (identity,status)=>{
  let action = [...actions()].filter(([key,value])=>(key.test(`${identity}_${status}`)))
  action.forEach(([key,value])=>value.call(this))
}
```

That is to say, by utilizing the characteristics of array looping, logic that meets the regex condition will be executed, allowing both common logic and individual logic to be executed at the same time. With the presence of regex, you can unleash your imagination to unlock more possibilities.
