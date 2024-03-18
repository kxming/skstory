---
title: 'JS-Web-API Knowledge Points'
date: 2024-01-27T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/0*OZL33xhjFl0XFUmv"
categories: ["Javascript"]
tags: ["Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "In addition to the ES basics, the web front-end often uses some browser-related APIs. Let’s sort them out together...."
---

# Knowledge Points

- BOM operations

- DOM operations

- Event binding

- Ajax

- Storage

---

## BOM

BOM (Browser Object Model) is the setting and obtaining of some information of the browser itself, such as acquiring the browser’s width and height, and setting where the browser should navigate to.

- navigator

- screen

- location

- history

These objects are just a bunch of simple and blunt API, which have no technical content and are not interesting to talk about. Everyone can understand them by looking them up on websites like MDN or W3school. During interviews, interviewers generally won’t ask too many questions about this, because as long as you have a good grasp of the basic knowledge, you can understand these APIs even if you can’t remember them, by just looking them up online. Below are some examples of commonly used functions.

Obtain browser features (also known as UA) and then recognize the client, for example, to determine whether it is a Chrome browser

```
var ua = navigator.userAgent
var isChrome = ua.indexOf('Chrome')
console.log(isChrome)
```

Obtain the width and height of the screen

```
console.log(screen.width)
console.log(screen.height)
```

Get url, protocol, path, parameters, hash etc

```
// For example, the current url is https://juejin.im/timeline/frontend?a=10&b=10#some
console.log(location.href)  // https://juejin.im/timeline/frontend?a=10&b=10#some
console.log(location.protocol) // https:
console.log(location.pathname) // /timeline/frontend
console.log(location.search) // ?a=10&b=10
console.log(location.hash) // #some
```

In addition, there are also functions to invoke the browser’s forward and backward functions, etc.

```
history.back()
history.forward()
```

## DOM

> Topic: What is the difference and connection between DOM and HTML?

### What is DOM

When talking about DOM, let’s first start with HTML, and when talking about HTML, let’s start with XML. XML is an extensible markup language. Being ‘extensible’ means that it can describe any kind of structured data. It is a tree!

```
<?xml version="1.0" encoding="UTF-8"?>
<note>
  <to>Tove</to>
  <from>Jani</from>
  <heading>Reminder</heading>
  <body>Don't forget me this weekend!</body>
  <other>
    <a></a>
    <b></b>
  </other>
</note>
```

HTML is an XML format with a predefined set of tags. The names, hierarchical relationships, and properties of the tags have been standardized (otherwise, they cannot be parsed by the browser). Similarly, it is also a tree.

```
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Document</title>
</head>
<body>
    <div>
        <p>this is p</p>
    </div>
</body>
</html>
```

The HTML code that we have developed is saved to a document (usually ending with .html or .htm). The document is stored on the server, and when the browser requests the server, this document is returned. Therefore, what the browser ultimately gets is just a document whose content is HTML-formatted code.

However, the browser needs to render the HTML in this document into a page according to the standard. At this point, the browser needs to process this bunch of code into something it can understand, and it also needs to process the code into something JS can understand, because it also allows JS to modify the page content.

Based upon these requirements, the browser needs to transform HTML into DOM. HTML is a tree, and DOM is also a tree. To understand DOM, you can temporarily set aside the internal factors of the browser and start with JS. In other words, you can regard DOM as the HTML structure that JS can recognize, a regular JS object or array.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*c_LfDgOZIWEIaktrBcdTLQ.png)

### Obtain DOM Nodes

The most commonly used DOM API is to obtain nodes, and the common methods of getting them are as shown in the code example below:

```
// Get by id
var div1 = document.getElementById('div1') // element
// Get by tagname
var divList = document.getElementsByTagName('div')  // collection
console.log(divList.length)
console.log(divList[0])
// Get by class
var containerList = document.getElementsByClassName('container') // collection
// Get by CSS selector
var pList = document.querySelectorAll('p') // collection
```

> Topic: What is the difference between properties and attributes?

### Property

A DOM node is a JS object, which complies with the characteristics of the object we mentioned before — extendable properties, because a DOM node is essentially a JS object. Therefore, as shown in the following code, p can have a style property, and it also has className, nodeName, nodeType properties. Note, these are all properties from the JS context, and they comply with the JS syntax standard.

```
var pList = document.querySelectorAll('p')
var p = pList[0]
console.log(p.style.width)  // Get style
p.style.width = '100px'  // Modify style
console.log(p.className)  // Get class
p.className = 'p1'  // Modify class
// Get nodeName and nodeType
console.log(p.nodeName)
console.log(p.nodeType)
```

### Attribute

The process of getting and modifying properties is changing the JS object directly, whereas the attribute is changing the HTML attribute directly, the two are quite different. The attribute is the get and set of HTML attributes, which has no relation to the property of the JS context of the DOM node.

```
var pList = document.querySelectorAll('p')
var p = pList[0]
p.getAttribute('data-name')
p.setAttribute('data-name', 'juejin')
p.getAttribute('style')
p.setAttribute('style', 'font-size:30px;')
```

Moreover, when you get and set attributes, it will trigger the query, reflow or repaint of the DOM. Frequent operations may affect page performance.

> Topic: What are the basic APIs for DOM operations?

### DOM Tree Manipulations

Adding Nodes

```
var div1 = document.getElementById('div1')
// Add new nodes
var p1 = document.createElement('p')
p1.innerHTML = 'this is p1'
div1.appendChild(p1) // Add the newly created element
// Move existing nodes. Note that this is "moving" and not "copying".
var p2 = document.getElementById('p2')
div1.appendChild(p2)
```

Accessing Parent Element

```
var div1 = document.getElementById('div1')
var parent = div1.parentElement
```

Accessing Child Elements

```
var div1 = document.getElementById('div1')
var child = div1.childNodes
```

Deleting Nodes

```
var div1 = document.getElementById('div1')
var child = div1.childNodes
div1.removeChild(child[0])
```

There are also APIs for other operations, such as accessing the previous node, accessing the next node, etc., but what is often asked in interviews are the above few.

## Events

### Event Binding

The traditional way of event binding is as follows:

```
var btn = document.getElementById('btn1')
btn.addEventListener('click', function (event) {
    // event.preventDefault() // Prevent default behavior
    // event.stopPropagation() // Prevent bubbling
    console.log('clicked')
})
```

In order to simplify event binding, you can create a universal event binding function. Although it’s simple, it will be improved and enriched as we go on.

```
// Universal event binding function
function bindEvent(elem, type, fn) {
    elem.addEventListener(type, fn)
}
var a = document.getElementById('link1')
// It's simpler to write
bindEvent(a, 'click', function(e) {
    e.preventDefault() // Prevent default behavior
    alert('clicked')
})
```

Lastly, if asked about IE lower version compatibility in an interview, I advise you to give up the job opportunity decisively. Nowadays, all internet traffic is on Apps, and the proportion of IE is getting less and less. It’s not worth to waste your youth for IE anymore. Try to pursue App-related work if possible.

> Topic: What is event bubbling?

### Event Bubbling

```
<body>
    <div id="div1">
        <p id="p1">Activate</p>
        <p id="p2">Cancel</p>
        <p id="p3">Cancel</p>
        <p id="p4">Cancel</p>
    </div>
    <div id="div2">
        <p id="p5">Cancel</p>
        <p id="p6">Cancel</p>
    </div>
</body>
```

Given the above HTML code structure, when p1 is clicked, it should be in the activated state. Clicking on any other `<p>` should cancel the activated state. How can we achieve this? The code is as follows, pay attention to the comments:

```
var body = document.body
bindEvent(body, 'click', function (e) {
    // All p clicks will bubble up to the body, because in the DOM structure, the body is an upper-level node of p, and events will bubble up along the DOM tree
    alert('Cancel')
})
var p1 = document.getElementById('p1')
bindEvent(p1, 'click', function (e) {
    e.stopPropagation() // Prevent bubbling
    alert('Activate')
})
```

If we bind events to p1 div1 body, they will bubble according to the DOM structure, executing one by one from bottom to top. But we can prevent bubbling by using e.stopPropagation().

> Topic: How do you use event delegation? What are the advantages?

### Event Delegation

Let’s set a scene where we have a `<div>` containing a number of `<a>` elements, and they can continue to increase. So how do we quickly and conveniently bind events to all the `<a>` elements?

```
<div id="div1">
    <a href="#">a1</a>
    <a href="#">a2</a>
    <a href="#">a3</a>
    <a href="#">a4</a>
</div>
<button>Click to add an a tag</button>
```

This is where event delegation comes in. We want to listen for `<a>` events, but we want to bind the specific events to the `<div>`, and then see if the event trigger point is an `<a>`.

```
var div1 = document.getElementById('div1')
div1.addEventListener('click', function (e) {
    // e.target can listen to find out which element triggered the click event
    var target = e.target
    if (e.nodeName === 'A') {
        // The <a> element was clicked
        alert(target.innerHTML)
    }
})
```

Let’s now improve the universal event binding function we wrote before and add event delegation.

```
function bindEvent(elem, type, selector, fn) {
    // With this process, we can accept two ways of calling: bindEvent(div1, 'click', 'a', function () {...}) and bindEvent(div1, 'click', function () {...})
    if (fn == null) {
        fn = selector
        selector = null
    }
// Bind event
    elem.addEventListener(type, function (e) {
        var target
        if (selector) {
            // If we have a selector, event delegation is needed
            // Get the element that triggered the event, i.e. e.target
            target = e.target
            // Check if it matches the selector condition
            if (target.matches(selector)) {
                fn.call(target, e)
            }
        } else {
            // If there's no selector, event delegation is not needed
            fn(e)
        }
    })
}
```

Then you can use it like this, which is much simpler.

```
// Use delegation, bindEvent gets an extra 'a' argument
var div1 = document.getElementById('div1')
bindEvent(div1, 'click', 'a', function (e) {
    console.log(this.innerHTML)
})
// Not using delegation
var a = document.getElementById('a1')
bindEvent(div1, 'click', function (e) {
    console.log(a.innerHTML)
})
```

Lastly, the benefits of using delegation are as follows:

- It simplifies the code

- It reduces memory consumption by the browser

## Ajax

### XMLHttpRequest

> Topic: Hand-written XMLHttpRequest without the help of any libraries

This is a common tool used by quirky and unique interviewers. There’s a lot of debate about this kind of test, but you can’t say it’s completely wrong as it tests your understanding of basic knowledge.

```
var xhr = new XMLHttpRequest()
xhr.onreadystatechange = function () {
    // The function here executes asynchronously, refer to the asynchronous module in JS basics
    if (xhr.readyState == 4) {
        if (xhr.status == 200) {
            alert(xhr.responseText)
        }
    }
}
xhr.open("GET", "/api", false)
xhr.send(null)
```

Of course, it’s much simpler to write with libraries like jQuery, Zepto, or Fetch, so I won’t elaborate on that.

### Explanation of Status Codes

There are two places where status codes need to be explained in the above code. xhr.readyState is used by the browser to judge the different stages of the request process, and xhr.status are the status codes for different results stipulated in the HTTP protocol.

Explanation of xhr.readyState status codes:

0 — Proxy created but open() method has not been called.

1 — open() method has been called.

2 — send() method has been called, and headers and status can be obtained.

3 — Downloading, responseText property already contains partial data.

4 — Download operation is complete

> Topic: In the HTTP protocol, what are the common status codes for a response?

xhr.status is the HTTP status code, there are 2xx, 3xx, 4xx, 5xx, the commonly used ones include:

- 200 Normal

- 301 Permanent redirect. For example, the GET request http://xxx.com (no / at the end) will be 301 redirected to http://xxx.com/ (with a / at the end).

- 302 Temporary redirect. It's temporary, not permanent.

- 304 Resource found but not meeting the request condition, will not return any body. For instance, when sending a GET request, if the head contains If-Modified-Since: xxx (requesting the return of resources updated after xxx time), and the server-side resource has not been updated, it will return 304, which means it doesn't meet the request requirements.

- 404 Resource not found

- 5xx Server side error

At the end, you should understand why the code above requires both xhr.readyState == 4 and xhr.status == 200 to be satisfied.

### Fetch API

Currently, there is a more convenient API for obtaining HTTP requests: Fetch. The fetch() global function provided by Fetch can easily initiate asynchronous requests and supports callback of Promise. However, the Fetch API is relatively new, and you would need to check caniuse for browser compatibility.

Here’s a simple example:

```
fetch('some/api/data.json', {
  method:'POST', // Request type GET, POST
  headers:{}, // Request headers, presented as Headers object or ByteString
  body:{}, // Request data blob, BufferSource, FormData, URLSearchParams (cannot include body in GET or HEAD methods)
  mode:'', // Request mode, if it involves cross-domain, like cors, no-cors or same-origin
  credentials:'', // Cross-domain policy for cookies, like omit, same-origin or include
  cache:'', // Request's cache mode: default, no-store, reload, no-cache, force-cache or only-if-cached
}).then(function(response) { ... });
```

Fetch supports headers definition, through customized headers, we can conveniently implement various request methods (PUT, GET, POST, etc.), headers (including cross-domain) and cache strategies, etc. In addition, it also supports various types of response (return data), like binary files, strings, and formData, etc.

### Cross-domain

> Topic: How to implement cross-domain?

There is a same-origin policy in the browser, that is, a page under a domain cannot use Ajax to get the interface of another domain. For example, there is an interface http://m.juejin.com/course/ajaxcourserecom?cid=459. Your own page http://www.yourname.com/page1.html cannot access this interface via Ajax. This is due to the "same-origin policy". If the browser ignores the same-origin policy somewhere, then it is a security loophole of the browser, which needs to be urgently fixed.

Where does a URL differ to be considered as cross-domain?

- Protocol

- Domain name

- Port

However, a few HTML tags can evade the same-origin policy — — `<script src="xxx">`, `<img src="xxxx"/>`, `<link href="xxxx">`. The src/href of these three tags can load resources from other domains, and are not restricted by the same-origin policy.

Therefore, these three tags can do some special things.

- `<img>` can be used for point statistics, because the statistical party does not necessarily come from the same domain. We have provided a code example when talking about the asynchronous JS basics. Besides its ability to cross-domain, `<img>` almost has no browser compatibility issue, because it is a very old tag.

- `<script>` and `<link>` can utilize CDN, and CDN links are generally from other domains.

- In addition, `<script>` can also implement JSONP, which can get the information from other domain interfaces. We will explain it right away.

But please note that for all cross-domain request methods, ultimately, the information provider needs to provide corresponding support and modifications, which means, only with the consent of the information provider can the receiver get their information. The browser will not allow otherwise.

### Resolve Cross-domain — JSONP

Firstly, you should understand the concept. For example, when visiting http://coding.m.juejin.com/classindex.html, does the server definitely have a classindex.html file? —— Not necessarily, the server can receive this request, dynamically generate a file, and then return it.
Similarly, `<script src="http://coding.m.juejin.com/api.js">` does not necessarily load a static file from the server, the server can also dynamically generate files and return them. OK, let's officially start.

For example, our website and Juejin’s site are definitely not the same domain. We need Juejin to provide an interface for us to get. Firstly, we define on our own pages like this:

```
<script>
window.callback = function (data) {
    // This is the information we get cross-domain
    console.log(data)
}
</script>
```

Next, Juejin provided me an http://coding.m.juejin.com/api.js, the content is as follows (as mentioned before, the server can dynamically generate content)

```
callback({x:100, y:200})
```

Lastly, we add `<script src="http://coding.m.juejin.com/api.js"></script>` on our page, and when this js is loaded, it will execute its content, and we will get the content.

### Resolve Cross-domain — Server Side Setting of HTTP Header

This needs to be set on the server side, as a front-end engineer, we don’t need to master the details, but we should know there is such a solution. Moreover, the now recommended cross-domain solution is this one, which is much simpler than JSONP.

```
response.setHeader("Access-Control-Allow-Origin", "http://m.juejin.com/");  // Fill in the domain name that allows cross-domain in the second parameter, it is not recommended to write "*"
response.setHeader("Access-Control-Allow-Headers", "X-Requested-With");
response.setHeader("Access-Control-Allow-Methods", "PUT,POST,GET,DELETE,OPTIONS");
// Accepting cookie cross-domain
response.setHeader("Access-Control-Allow-Credentials", "true");
```

## Storage

> Topic: What is the difference between cookies and localStorage?

### Cookies

Cookies are not originally designed for server-side storage (there are many examples of such “cats chasing dogs” in the computer world, such as float in CSS). They are designed to convey information between servers and clients, so all our HTTP requests carry cookies. However, cookies also have the capability of browser-side storage (such as remembering usernames and passwords), so they have been used by developers.

They are very simple to use, just document.cookie = .....

But cookies have their fatal disadvantages:

- The storage capacity is too small, only 4KB

- All HTTP requests carry them, affecting the efficiency of resource acquisition

- The API is simple and needs to be encapsulated to be used

### LocalStorage and sessionStorage

Later, HTML5 brought sessionStorage and localStorage. Let's talk about localStorage first. It was specifically designed for browser-side caching. Its advantages include:

- Its storage capacity is increased to 5MB

- It is not included in HTTP requests

- The API is suitable for data storage, which include localStorage.setItem(key, value), localStorage.getItem(key)

The difference for sessionStorage is that it is based on the session expiration time, while localStorage is permanently valid, so they are used in different scenarios. For example, some important information that needs to be invalidated in time is put into sessionStorage, and some unimportant but infrequently set information is put into localStorage.

Another tip for you, when using localStorage.setItem, you should try to add it into try-catch, as some browsers disable this API, so be attentive.
