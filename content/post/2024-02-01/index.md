---
title: 'Little-known but useful js tricks【one line of code】'
date: 2024-02-01T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/0*dpgmDdOfYm727sko"
categories: ["Javascript","Function"]
tags: ["Javascript","Function"]
URL: ""
layout: posts
is_recommend: true
description: ""
---

## Array

### Generating an array

When you need to generate an array from 0 to 99

option 1

```
const createArr = (n) => Array.from(new Array(n), (v, i) => i)
const arr = createArr(100)
```

option 2

```
const createArr = (n) => new Array(n).fill(0).map((v, i) => i)
createArr(100)
```

### Shuffling an array

When you have an array that you need to randomize

```
const randomSort = list => list.sort(() => Math.random() - 0.5)
randomSort([0,1,2,3,4,5,6,7,8,9])
```

### Array simple data deduplication

When you need to remove all duplicate elements in an array, leaving only one

```
const removeDuplicates = list => [...new Set(list)]
removeDuplicates([0, 0, 2, 4, 5])
```

### Array unique value data deduplication

When you need to remove duplicate elements in an array based on a unique value

```
const duplicateById = list => [...list.reduce((prev, cur) => prev.set(cur.id, cur), new Map()).values()]
duplicateById([{id: 1, name: 'jack'}, {id: 2, name: 'rose'}, {id: 1, name: 'jack'}])
// [{id: 1, name: 'jack'}, {id: 2, name: 'rose'}]
```

### Finding array intersection

When you need to find the intersection of multiple arrays

```
const intersection = (a, ...arr) => [...new Set(a)].filter((v) => arr.every((b) => b.includes(v)))

intersection([1, 2, 3, 4], [2, 3, 4, 7, 8], [1, 3, 4, 9])
// [3, 4]
```

### Finding the index of the maximum value

When you need to find the index of the maximum value in an array

```
const indexOfMax = (arr) => arr.reduce((prev, curr, i, a) => (curr > a[prev] ? i : prev), 0);
indexOfMax([1, 3, 9, 7, 5]); // 2
```

### Finding the index of the minimum value

When you need to find the index of the minimum value in an array

```
const indexOfMin = (arr) => arr.reduce((prev, curr, i, a) => (curr < a[prev] ? i : prev), 0)
indexOfMin([2, 5, 3, 4, 1, 0, 9]) // 5
```

### Finding the nearest value

When you need to find the nearest value in an array

```
const closest = (arr, n) => arr.reduce((prev, curr) => (Math.abs(curr - n) < Math.abs(prev - n) ? curr : prev))
closest([29, 87, 8, 78, 97, 20, 75, 33, 24, 17], 50) // 33
```

### Compressing multiple arrays

When you need to compress multiple arrays into one

```
const zip = (...arr) => Array.from({ length: Math.max(...arr.map((a) => a.length)) }, (_, i) => arr.map((a) => a[i]))
zip([1,2,3,4], ['a', 'b', 'c', 'd'], ['A', 'B', 'C', 'D'])
// [[1, 'a', 'A'], [2, 'b', 'B'], [3, 'c', 'C'], [4, 'd', 'D']]
```

### Matrix row and column exchange

When you need to swap the rows and columns of a matrix

```
const transpose = (matrix) => matrix[0].map((col, i) => matrix.map((row) => row[i]));
transpose(
    [              // [
        [1, 2, 3], //      [1, 4, 7],
        [4, 5, 6], //      [2, 5, 8],
        [7, 8, 9], //      [3, 6, 9],
     ]             //  ]
 ); 
```

### Sort the array by a specific attribute

```
// Input: 'name', [{name:'Bob', age:25}, {name:'Alice', age:22}]
const sortBy = (arr, key) => arr.sort((a, b) => a[key] > b[key] ? 1 : -1);
// Output: [{name:'Alice', age:22}, {name:'Bob', age:25}]
```

### Check if a variable is an array

The Array.isArray() method checks if a given variable is an array.

```
const isArray = variable => Array.isArray(variable);
```

### Retrieve the last item in an array

```
const lastItem = array => array.slice(-1)[0];
```

### Capitalize the first letter of a string

```
const capitalize = string => string.charAt(0).toUpperCase() + string.slice(1);
```

## Number conversion

### Radic conversion
To convert decimal to n-base, you can use toString(n)

```
const toDecimal = (num, n = 10) => num.toString(n) 
// The decimal number 10 needs to be converted into binary.
toDecimal(10, 2) // '1010'
```

To convert n-base to decimal, you can use parseInt(num, n)

```
const toDecimalism = (num, n = 10) => parseInt(num, n)
toDecimalism(1010, 2)
```

### Regular Expression

#### Phone number formatting

When you need to format a phone number in the form of xxx-xxxx-xxxx

```
const formatPhone = (str, sign = '-') => str.replace(/(\W|\s)/g, "").split(/^(\d{3})(\d{4})(\d{4})$/).filter(item => item).join(sign)

formatPhone('13123456789') // '131-2345-6789'
formatPhone('13 1234 56 789', ' ') // '131 2345 6789'
```

#### Remove excess spaces
When you need to merge multiple spaces in a text into a single space

```
const setTrimOut = str => str.replace(/\s\s+/g, ' ')
const str = setTrimOut('hello,   jack') // 
```

## Web

### Reload the current page

```
const reload = () => location.reload();
reload()
```

### Scroll to the top of the page

If you need to scroll the page to the top

```
const goToTop = () => window.scrollTo(0, 0);
goToTop()
```

### Element scrolling

If you want to smoothly scroll an element to the start of the viewport

```
const scrollToTop = (element) =>
  element.scrollIntoView({ behavior: "smooth", block: "start" })
scrollToTop(document.body)
```

If you want to smoothly scroll an element to the end of the viewport

```
const scrollToBottom = (element) =>
  element.scrollIntoView({ behavior: "smooth", block: "end" })
  scrollToBottom(document.body)
```

### Check if the current browser is IE

```
const isIE = !!document.documentMode;
```

### Stripping HTML from given text

When you need to filter out all the tags in a piece of text

```
const stripHtml = (html) => new DOMParser().parseFromString(html, 'text/html').body.textContent || '';
stripHtml('<div>test</div>') // 'test'
```

### Redirect

When you need to jump to another page

```
const goTo = (url) => (location.href = url);
```

### Text pasting

When you need to copy text to the clipboard

```
const copy = (text) => navigator.clipboard?.writeText && navigator.clipboard.writeText(text)
copy('copy text')
```

### Copy content to clipboard

```
const copyToClipboard = (text) => navigator.clipboard.writeText(text);
copyToClipboard("Hello World");
```

### Clear all cookies

```
const clearCookies = document.cookie.split(';').forEach(cookie => document.cookie = cookie.replace(/^ +/, '').replace(/=.*/, `=;expires=${new Date(0).toUTCString()};path=/`));
```

### Get the selected text

```
const getSelectedText = () => window.getSelection().toString();
getSelectedText();
```

### Detect if it’s dark mode

```
const isDarkMode = window.matchMedia && window.matchMedia('(prefers-color-scheme: dark)').matches
console.log(isDarkMode)
```

### Determine if the current tab is active

```
const isTabInView = () => !document.hidden;
```

### Determine if it is currently an Apple device

```
const isAppleDevice = () => /Mac|iPod|iPhone|iPad/.test(navigator.platform);
isAppleDevice();
```

### Open the browser print dialog

```
const showPrintDialog = () => window.print()
```

### Check if an element is in the viewport

```
const elementInViewport = el => el.getBoundingClientRect().top >= 0 && el.getBoundingClientRect().bottom <= window.innerHeight;
```

### Get device type

```
const getDeviceType = () => /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(navigator.userAgent) ? 'Mobile' : 'Desktop';
```

## Date

### Check if a date is today

```
const isToday = (date) => date.toISOString().slice(0, 10) === new Date().toISOString().slice(0, 10)
```

### Date conversion

When you need to convert a date into YYYY-MM-DD format

```
const formatYmd = (date) => date.toISOString().slice(0, 10);
formatYmd(new Date())
```

### Time conversion

When you need to convert seconds into hh:mm:ss format

```
const formatSeconds = (s) => new Date(s * 1000).toISOString().substr(11, 8)
formatSeconds(200) // 00:03:20
```

### Get the first day of a month in a particular year

When you need to get the first day of a month in a specific year

```
const getFirstDate = (d = new Date()) => new Date(d.getFullYear(), d.getMonth(), 1);
getFirstDate(new Date('2022-04')) // Fri Apr 01 2022 00:00:00 GMT+0800 (中国标准时间)
```

### Get the last day of a month in a particular year

When you need to get the last day of a month in a specific year

```
const getLastDate = (d = new Date()) => new Date(d.getFullYear(), d.getMonth() + 1, 0);
getLastDate(new Date('2023-03-04')) // Fri Mar 31 2023 00:00:00 GMT+0800 (中国标准时间)
```

### Get the total number of days in a month in a particular year

When you need to get the total number of days in a specific month of a certain year

```
const getDaysNum = (year, month) => new Date(year, month, 0).getDate()  
const day = getDaysNum(2024, 2) // 29
```

### Check if a date is valid

```
const isDateValid = (...val) => !Number.isNaN(new Date(...val).valueOf());
isDateValid("December 17, 1995 03:24:00"); 
```

### Calculate the interval between two dates

```
const dayDif = (date1, date2) => Math.ceil(Math.abs(date1.getTime() - date2.getTime()) / 86400000)
dayDif(new Date("2021-11-3"), new Date("2022-2-1")) 
```

### Find out which day of the year a date falls on

```
const dayOfYear = (date) => Math.floor((date - new Date(date.getFullYear(), 0, 0)) / 1000 / 60 / 60 / 24);
dayOfYear(new Date());
```

### Obtain the timezone

```
Intl.DateTimeFormat().resolvedOptions().timeZone;
```

## Function

### Asynchronous function judgment

Determine whether a function is an asynchronous function

```
const isAsyncFunction = (v) => Object.prototype.toString.call(v) === '[object AsyncFunction]'
isAsyncFunction(async function () {}); // true
```

## Number

### Truncate number

When you need to truncate some digits after the decimal point without rounding

```
const toFixed = (n, fixed) => `${n}`.match(new RegExp(`^-?\d+(?:.\d{0,${fixed}})?`))[0]
toFixed(10.255, 2) // 10.25
```

### Rounding

When you need to truncate some digits after the decimal point and round

```
const round = (n, decimals = 0) => Number(`${Math.round(`${n}e${decimals}`)}e-${decimals}`)
round(10.255, 2) // 10.26
```

### Zero-padding

When you need to pad zeros in front of a number num that lacks len digits

```
const replenishZero = (num, len, zero = 0) => num.toString().padStart(len, zero)
replenishZero(8, 2) // 08
```

## Object

### Remove invalid properties

When you need to remove all properties in an object with a property value of null or undefined

```
const removeNullUndefined = (obj) => Object.entries(obj).reduce((a, [k, v]) => (v == null ? a : ((a[k] = v), a)), {});
removeNullUndefined({name: '', age: undefined, sex: null}) // { name: '' }
```

### Reverse object key-value pairs

When you need to exchange the key-value pairs of an object

```
const invert = (obj) => Object.keys(obj).reduce((res, k) => Object.assign(res, { [obj[k]]: k }), {})
invert({name: 'jack'}) // {jack: 'name'}
```

### Check if an object is empty

```
const isEmpty = obj => Reflect.ownKeys(obj).length === 0 && obj.constructor === Object;
```

### String to object

```
const strParse = (str) => JSON.parse(str.replace(/(\w+)\s*:/g, (_, p1) => `"${p1}":`).replace(/\'/g, "\""))
strParse('{name: "jack"}')
```

## Other

### Compare two objects

When you need to compare two objects. JavaScript’s equality operator can only determine if the addresses of two objects are the same, and cannot determine if the key-value pairs of two objects are consistent when the addresses are different.

```
const isEqual = (...objects) => objects.every(obj => JSON.stringify(obj) === JSON.stringify(objects[0]))
isEqual({name: 'jack'}, {name: 'jack'}) // true
isEqual({name: 'jack'}, {name: 'jack1'}, {name: 'jack'}) // false
```

### Generate random color

When you need to generate a random color

```
const getRandomColor = () => `#${Math.floor(Math.random() * 0xffffff).toString(16)}`
getRandomColor() // '#4c2fd7'
```

### Color format conversion

When you need to convert hexadecimal color to rgb

```
const hexToRgb = hex => hex.replace(/^#?([a-f\d])([a-f\d])([a-f\d])$/i, (_, r, g, b) => `#${r}${r}${g}${g}${b}${b}`).substring(1).match(/.{2}/g).map((x) => parseInt(x, 16));
hexToRgb('#00ffff'); // [0, 255, 255]
hexToRgb('#0ff'); // [0, 255, 255]
```

### Generate random IP address

When you need to generate an IP address

```
const randomIp = () =>
    Array(4)
        .fill(0)
        .map((_, i) => Math.floor(Math.random() * 255) + (i === 0 ? 1 : 0))
        .join('.');
```

### UUID generation

When you need to generate an id

```
const uuid = (a) => (a ? (a ^ ((Math.random() * 16) >> (a / 4))).toString(16) : ([1e7] + -1e3 + -4e3 + -8e3 + -1e11).replace(/[018]/g, uuid))
uuid()
```

### Get cookie

When you need to convert cookie into an object

```
const getCookie = () => document.cookie
    .split(';')
    .map((item) => item.split('='))
    .reduce((acc, [k, v]) => (acc[k.trim().replace('"', '')] = v) && acc, {})
getCookie()
```

### Force waiting

When you need to wait for a period of time, but you don’t want to write it in the setTimeout function, causing callback hell.

```
const sleep = async (t) => new Promise((resolve) => setTimeout(resolve, t));
sleep(2000).then(() => {console.log('time')});
```

### Conversion between Fahrenheit and Celsius

```
const celsiusToFahrenheit = (celsius) => celsius * 9/5 + 32;

const fahrenheitToCelsius = (fahrenheit) => (fahrenheit - 32) * 5/9;
```
