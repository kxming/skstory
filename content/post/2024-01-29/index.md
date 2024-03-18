---
title: 'Handle Algorithmic Questions Calmly'
date: 2024-01-29T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/0*OZL33xhjFl0XFUmv"
categories: ["Javascript"]
tags: ["Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "Engineers in computer-related fields must master certain knowledge of data structures and algorithms...."
---

We know from the composition of von Neumann machines that data storage and computation are the main tasks of a computer. `Program = Data Structure + Algorithm`, thus, engineers in computer-related fields must master certain knowledge of data structures and algorithms.

# Knowledge Summary

- Common Data Structures

  - Stacks, Queues, Linked Lists

  - Sets, Dictionaries, Hash Sets

- Common Algorithms

  - Recursion

  - Sorting

  - Enumeration

- Algorithm Complexity Analysis

- Algorithmic Thinking

  - Divide and Conquer

  - Greedy

  - Dynamic Programming

- Advanced Data Structure

  - Trees, Graphs

  - Depth-First and Breadth-First Searches

This section will guide everyone to quickly review data structures and algorithms, focusing on frequently tested algorithms and data structures used in front-end development.

---

## Data Structure

Data structure determines the spatial and temporal efficiency of data storage. The requirements of data write and read speed also decide what type of data structure should be chosen.

Depending on the different needs of the scene, we design different data structures, such as:

- Data structures with more reads: Try to increase the efficiency of data reads, such as IP databases, which only need to be written once, and the rest are all reads.

- Data structures with more reads and writes: Balance both needs, such as the LRU Cache algorithm.

Algorithm is the way to process the data, a determined algorithm can improve the efficiency of data processing. For example, binary search in an ordered array is much faster than ordinary sequential search, especially when handling large amounts of data.

Data structures and algorithms are universal skills in program development, so they may be encountered in any interview. With AI, big data, mini-games becoming more popular in recent years, Web developers inevitably have encounters with data structures and algorithms, and there will be more and more algorithmic problems in interviews. Studying data structures and algorithms also helps us to broaden our thinking and break through skill bottlenecks.

### Frequently seen Data Structure problems in front-end development

Now, let’s summarize the common data structures in front-end development:

- Simple Data Structures (must be understood and mastered)

- Ordered Data Structures: Stacks, Queues, Linked Lists, ordered data structures save space (storage space is small)

- Unordered Data Structures: Sets, Dictionaries, Hash Tables, unordered data structures save time (read time is fast)

- Complex Data Structures

- Trees, Heaps

- Graphs

For simple data structures, they correspond to arrays (Array) and objects (Object) in ES. You can think about it, the storage of arrays is ordered, while that of objects is unordered, but if I want to find a value key in the object, it is returned immediately, whereas there is a search process for arrays.

Here, I will introduce the design of data structures through a real interview question.

> Topic: Implement an event class Event using ECMAScript (JS) code, which includes the following features: binding events, unbinding events, and dispatching events.

In slightly more complex pages, such as those developed with components, the same page is developed by two or three people. In order to maintain the independence of the components and reduce the coupling between the components, we often use the ‘subscribe and publish model’. That is, the communication between components is done by listening for events and dispatching them, rather than directly calling the methods of each other. This is the Event class required by the subject.

The essence of this question is the data design of corresponding callback functions for each event type. In order to implement event binding, we need a _cache object to record which events are bound. When an event occurs, we need to get event callbacks from _cache and execute them one by one. Generally, event dispatching (read) happens more often than event binding (write) in a webpage. Therefore, the data structure we design should be able to find the corresponding event callback functions faster when an event occurs, and then execute them.

With this in mind, I simply wrote a sample code implementation:

```
class Event {
    constructor() {
        // The data structure that stores the events
        // Using a dictionary (object) for swift access
        this._cache = {};
    }
    // Bind
    on(type, callback) {
        // For easy searching and to save space,
        // We put the same type of events in an array
        // The array here is a queue, adhering to the concept first in first out
        // That is, the earlier bound event would be triggered first
        let fns = (this._cache[type] = this._cache[type] || []);
        if (fns.indexOf(callback) === -1) {
            fns.push(callback);
        }
        return this;
    }
    // Trigger
    trigger(type, data) {
        let fns = this._cache[type];
        if (Array.isArray(fns)) {
            fns.forEach((fn) => {
                fn(data);
            });
        }
        return this;
    }
    // Unbind
    off(type, callback) {
        let fns = this._cache[type];
        if (Array.isArray(fns)) {
            if (callback) {
                let index = fns.indexOf(callback);
                if (index !== -1) {
                    fns.splice(index, 1);
                }
            } else {
                // Clear all
                fns.length = 0;
            }
        }
        return this;
    }
}
// Test case
const event = new Event();
event.on('test', (a) => {
    console.log(a);
});
event.trigger('test', 'hello world');

event.off('test');
event.trigger('test', 'hello world');
```

For advanced data structures like trees, heaps, and graphs, front-end usually don’t ask too many about them, but their search methods are often asked, which I will introduce later. For advanced data, you should accumulate more and understand them well in your regular times. For example, once you understand that what a heap is, you will solve algorithm problems like “find the maximum K numbers” encountered in the interview.

## The efficiency of algorithms is measured through algorithmic complexity

The quality of an algorithm can be measured by its complexity, which includes time complexity and space complexity. Time complexity is the focus of the interview because of its characteristics such as easy estimation and evaluation. Space complexity is not often asked about in interviews.

The common time complexities include:

- Constant Order O(1)

- Logarithmic Order O(logN)

- Linear Order O(n)

- Linear Logarithmic Order O(nlogN)

- Squared Order O(n^2)

- Cubic Order O(n^3)

- k-th Power Order O(n^k)

- Exponential Order O(2^n)

With the increasing size of the problem n, the time complexity increases correspondingly, and the execution efficiency of the algorithm decreases.

Generally, when analyzing the complexity of an algorithm, we follow these tips:

1. See how many loops there are. Generally speaking, one loop is O(n), two loops will be O(n^2), and so on.

2. If there is a binary operation, then it is O(logN).

3. Keep the highest term and discard the constant term.

> Topic: Analyze the algorithmic complexity of the following code (for clarity, I have added a code analysis in the comments)

```
let i =0; // statement is executed once
while (i < n) { // statement is executed n times
  console.log(`Current i is ${i}`); //statement is executed n times
  i++; // statement is executed n times
}
```

According to the comments, we can deduce that the algorithmic complexity is `1 + n + n + n = 1 + 3n`, discarding the constant term, it is O(n).

```
let number = 1; // statement is executed once
while (number < n) { // statement is executed logN times
  number *= 2; // statement is executed logN times
}
```

The exit condition for the loop in the above code is `number<n, and number grows at a speed of 2^n in the body of the loop. So, in reality, the loop code executes logN times, so the complexity is: 1 + 2 * logN = O(logN)`

```
for (let i = 0; i < n; i++) {// the statement is executed n times
  for (let j = 0; j < n; j++) {// the statement is executed n^2 times
    console.log('I am here!'); // the statement is executed n^2 times
  }
}
```

The complexity of the code with two nested for loops is easily deduced to be O(n^2).

## Basic Algorithms Everyone Should Master

Enumeration and recursion are the simplest algorithms and also the basis of complex algorithms, which everyone should master! Enumeration is relatively simple, so we will focus more on elaborating recursion.

Recursion consists of two parts:

1. The body of recursion, which is the code that needs to be iteratively solved.

2. The breakout condition of recursion: recursion cannot go on indefinitely, it must exit after meeting a certain condition.

There is a classic interview question about recursion:

> Topic: Implement deep copy of JavaScript objects.

**What is deep copy?**

Deep copy means that all reference structures of the data are duplicated when copying data. Simply put, it is the duplication of reference types of data rather than copying of their relationship only.

Analyze how to make a “deep copy”:

1. Assume that the method for deep copying has been completed, named deepClone

2. To copy a piece of data, we certainly need to iterate its properties. If a property of this object is still an object, continue using this method repeatedly.

```
function deepClone(o1, o2) {
    for (let k in o2) {
        if (typeof o2[k] === 'object') {
            o1[k] = {};
            deepClone(o1[k], o2[k]);
        } else {
            o1[k] = o2[k];
        }
    }
}
// Test case
let obj = {
    a: 1,
    b: [1, 2, 3],
    c: {}
};
let emptyObj = Object.create(null);
deepClone(emptyObj, obj);
console.log(emptyObj.a == obj.a);
console.log(emptyObj.b == obj.b);
```

Recursion can easily lead to stack overflow, which can be solved by tail call. V8 engine of Chrome has optimized tail call, and we should also take note of tail call syntax when writing code. The problem of stack overflow in recursion can be replaced by enumeration, which is to replace recursion with for or while loop.

When we use recursion, we need to optimize it, as demonstrated in the following example.

> Topic: Find the nth term in the Fibonacci sequence (rabbit sequence) 1,1,2,3,5,8,13,21,34,55,89…

In the following code, the count records the number of recursions, and we can compare the values of count used in two different codes:

```
let count = 0;
function fn(n) {
    let cache = {};
    function _fn(n) {
        if (cache[n]) {
            return cache[n];
        }
        count++;
        if (n == 1 || n == 2) {
            return 1;
        }
        let prev = _fn(n - 1);
        cache[n - 1] = prev;
        let next = _fn(n - 2);
        cache[n - 2] = next;
        return prev + next;
    }
    return _fn(n);
}
let count2 = 0;
function fn2(n) {
    count2++;
    if (n == 1 || n == 2) {
        return 1;
    }
    return fn2(n - 1) + fn2(n - 2);
}
console.log(fn(20), count); // 6765 20
console.log(fn2(20), count2); // 6765 13529
```

### Quick Sort and Binary Search

The probability of a sorting or searching question appearing in a front-end interview is relatively small because the JS engine has optimized these common operations. You might find it interesting that the sorting method you strenuously write may not be as fast and concise as Array.sort. Therefore, mastering quick sort and binary search is enough.

Both quick sort and binary search are based on a kind of algorithmic thinking called “divide and conquer”, which achieves O(logN) (logarithmic level, a level lower than O(n) linear complexity) complexity by classifying and processing data, constantly decreasing the order of magnitude. The quick sort core is the O(logN) of the binary method, and the actual complexity is O(N*logN).

### Quick Sort

The approximate process of quick sort is as follows:

1. Randomly select a number A from the array as the benchmark.

2. Compare other numbers with this number, put smaller numbers on its left and larger numbers on its right.

3. After one loop, the numbers on the left side of A are smaller than A and the numbers on the right side of A are larger than A.

4. At this point, recursively sort the numbers to the left and right of A in the above manner.

The specific code is as follows:

```
// Partition operation function
function partition(array, left, right) {
    // Use index to get the middle value instead of splice
    const pivot = array[Math.floor((right + left) / 2)];
    let i = left;
    let j = right;
    while (i <= j) {
        while (compare(array[i], pivot) === -1) {
            i++;
        }
        while (compare(array[j], pivot) === 1) {
            j--;
        }
        if (i <= j) {
            swap(array, i, j);
            i++;
            j--;
        }
    }
    return i;
}
// Comparison function
function compare(a, b) {
    if (a === b) {
        return 0;
    }
    return a < b ? -1 : 1;
}
function quick(array, left, right) {
    let index;
    if (array.length > 1) {
        index = partition(array, left, right);
        if (left < index - 1) {
            quick(array, left, index - 1);
        }
        if (index < right) {
            quick(array, index, right);
        }
    }
    return array;
}
function quickSort(array) {
    return quick(array, 0, array.length - 1);
}
// In-place swapping function, instead of using a temporary array.
function swap(array, a, b) {
    [array[a], array[b]] = [array[b], array[a]];
}
const Arr = [85, 24, 63, 45, 17, 31, 96, 50];
console.log(quickSort(Arr));
```

### Binary Search

Binary search is mainly used to solve problems like “finding a specific number from a group of ordered numbers”. Regardless of whether these numbers are in a one-dimensional or multi-dimensional array, as long as they are ordered, binary search can be used for optimization.

Binary search is an algorithm implementing the “divide and conquer” strategy. The rough process is as follows:

1. Compare the number A in the middle of the array with the number to be found.

2. Given the array is sorted, a) if A is higher, the target number should be found in the first half, b) if A is lower, it should be searched in the second half of the array.

3. In this way, keep searching while reducing the scale (discard half of the data), until the search across the array is completed.

> Topic: In a two-dimensional array, each row is sorted in ascending order from left to right, and each column is sorted in ascending order from top to bottom. Please complete a function, input such a two-dimensional array and an integer, and determine whether the array contains this number.

```
function Find(target, array) {
    let i = 0;
    let j = array[i].length - 1;
    while (i < array.length && j >= 0) {
        if (array[i][j] < target) {
            i++;
        } else if (array[i][j] > target) {
            j--;
        } else {
            return true;
        }
    }
    return false;
}
//Test case
console.log(Find(10, [
    [1, 2, 3, 4], 
    [5, 9, 10, 11], 
    [13, 20, 21, 23]
    ])
);
```

Additionally, the author has encountered the following question in an interview:

> Topic: Now I have a positive integer in the range from 1 to 1000, and you need to guess what this number is. You can only ask one question: is it too big or too small? How many times do I need to guess to get it right?

When got this question, the author thought of a “guess the price” shopping program on television. If you can guess the price correctly within a specified time, you can take the goods home. So the question is to let the interviewer constantly tell me whether the number I guess is larger or smaller than this number. This is binary search!

How many times to guess? Actually, this question is a problem of the time complexity of the binary search algorithm. The time complexity of binary search is O(logN), so finding the solution of log1000 is the number of guesses. We know that 2^10=1024, so we can quickly estimate that log1000 is about 10. I could find the number by asking no more than 10 times!

**What to do if you encounter an algorithmic problem you can’t solve in an interview**

During an interview, when you encounter algorithmic problems, you should speculate on the interviewer’s intentions, listen carefully to the keywords, such as: searching an ordered sequence, the required algorithm complexity is O(logN), which generally means using the binary search concept.

Generally, the solution to the algorithm problem is divided into the following four steps:

1. First, reduce the order of magnitude and come up with a problem-solving process with solvable situations (data).

2. Write a program according to the problem-solving steps, and prioritize the handling of special conditions, such as a problem with a large array, if the array length is of two numbers.

3. Check the correctness of the program.

4. Can it be optimized (from shallow to deep)? If possible, you can deliberately reserve optimization points to highlight personal technical abilities.

## Regular Expression Problem Solving

Many algorithm problems are simpler to answer by utilising features of ES syntax, such as regular expressions. The author will summarise some knowledge about regular expressions through a few real questions.

> Topic: The first character that appears only once in a string

Please implement a function to find the first character that appears only once in a character stream. For example, when the first two characters “go” is read from the character stream, the first character that appears only once is “g”. When the first six characters “google” is read from this character stream, the first character that appears only once is “l”.

If you use pure algorithm to answer this question, you need to traverse each string, count the number of times each character appears, and then find the first character that appears only once according to the order of the string. The whole process is quite cumbersome, but with regular expression, things get a lot simpler.

```
function find(str){
    for (var i = 0; i < str.length; i++) {
        let char = str[i]
        let reg = new RegExp(char, 'g');
        let l = str.match(reg).length
        if(l===1){
            return char
        }
    }
}
```

Of course, using indexOf/lastIndexOf is also a clever way. Now let's look at a thousand bit problem.

> Topic: Change 1234567 to 1,234,567, i.e., mark it in thousands

This problem can be directly solved with algorithms. If a candidate uses regular expressions to answer, thus actively showcasing his other abilities, the interviewer generally won’t make it too difficult for him, even if the answer is not derived from an algorithm. This problem can be solved with the “zero-width assertion” `(?=exp)` of regular expression, which asserts that the position where it appears can match the expression exp. The feature of the thousandth bit is that the number of numbers after the first comma is a multiple of 3, regular expression: `/(\d{3})+$/`; there can be `1~3` numbers at most before the first comma, regular expression: `/\d{1,3}/`. Together they make up `/ \d{1,3}(\d{3})+$/`, the separator needs to be added from front to back.

```
function exchange(num) {
    num += ''; //Convert to string
    if (num.length <= 3) {
        return num;
    }
    num = num.replace(/\d{1,3}(?=(\d{3})+$)/g, (v) => {
        console.log(v)
        return v + ',';
    });
    return num;
}
console.log(exchange(1234567));
```

Of course, most of the above mentionings are clever ways to answer algorithmic questions. The following problem is a pure regular expression examination. The author has encountered it in an interview and will mention it here.

> Topic: please write out the execution result of the following code

```
var str = 'google';
var reg = /o/g;
console.log(reg.test(str))
console.log(reg.test(str))
console.log(reg.test(str))
```

After executing the code, you will find that the last one is not true, but false. This is because reg has a g, the global property. In this case, lastIndex comes into play. If you look at the execution results of the following code, you will understand.

```
console.log(reg.test(str), reg.lastIndex)
console.log(reg.test(str), reg.lastIndex)
console.log(reg.test(str), reg.lastIndex)
```

In actual development, this error can also be committed. For example, to reduce the number of variable definitions, commonly used variables are defined in advance. In this way, when used, it’s easy to fall into a trap, like the following code:

```
(function(){
    const reg = /o/g;
    function isHasO(str){
        // reg.lastIndex = 0; This can avoid the situation
        return reg.test(str)
    }
    var str = 'google';
    console.log(isHasO(str))
    console.log(isHasO(str))
    console.log(isHasO(str))
}())
```

## Summary

Regular expressions are a tool that candidates can use to showcase their problem-solving capabilities in a less conventional way during an interview. By flexibly applying regular expressions, you can solve certain problems more efficiently. However, be careful of the peculiarities of certain regular expressions, such as the ‘g’ flag, to avoid unexpected results.

Regular expressions are a powerful tool, but also a complex one. Therefore, understanding key regular expression concepts and practicing typical use cases are useful ways to enhance your regular expression skills and apply them effectively in problem solving.

## Conclusion

Regular expressions are powerful in performing pattern matching and sophisticated text manipulations, and practice makes perfect. If you have any questions or better solutions, you are welcome to share and discuss. Let’s move forward together on the programming journey.
