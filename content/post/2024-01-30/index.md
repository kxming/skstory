---
title: 'Browser Related Knowledge'
date: 2024-01-30T16:22:38+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/0*OZL33xhjFl0XFUmv"
categories: ["Javascript"]
tags: ["Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "As the pages written by Web front-end engineers have to run in the browser, many browser-related interview questions will appear in the interview...."
---

As the pages written by Web front-end engineers have to run in the browser, many browser-related interview questions will appear in the interview.

# Knowledge Consolidation

- Browser loading and rendering process

- Performance optimization

- Web security

This section will start with the browser’s loading process, then introduce how to optimize performance, and finally introduce common security issues and precautions in Web development.

---

## Loading Pages and Rendering Process

The loading process and rendering process can be divided into two parts. When answering questions, the key is to grasp the core points, cover all the points, with a little analysis, and be concise without dragging.

> Topic: The process of browser loading and rendering pages

### Loading Process

The points are as follows:

- The browser gets the IP address of the domain name according to the DNS server

- Send an HTTP request to the machine of this IP

- The server receives, processes, and returns the HTTP request

- The browser gets the returned content

In fact, it is a bunch of HMTL formatted strings, because only the HTML format can be correctly parsed by the browser, which is required by W3C standards. Next is the browser’s rendering process.

### Rendering Process

The points are as follows:

- Generate the DOM tree based on the HTML structure

- Generate CSSOM based on CSS

- Integrate DOM and CSSOM to form the RenderTree

- Start rendering and displaying based on RenderTree

- When you encounter a `<script>`, you will execute and block rendering

In the previous paragraph, the browser has already obtained the HTML content returned from the server and starts parsing and rendering. The content initially obtained was simply a bunch of strings, they must first be structured into basic data structures that computers are good at processing, hence HTML strings are transformed to a DOM tree — tree is one of the most basic data structures.

During the parsing process, if you encounter tags like `<link href="...">` and `<script src="...">` that load CSS and JS externally, the browser will download them asynchronously, the download process is the same as when downloading HTML content above. However, the downloaded strings are in CSS or JS format here.

The browser generates CSSOM from CSS, then integrates DOM and CSSOM to form the RenderTree, and then it can start rendering based on RenderTree. Think about it, if you have the DOM structure and styles, you can meet the rendering conditions at this time. In addition, this can also explain a question. **Why should CSS be placed in the HTML header?** This will allow the browser to get CSS as early as possible to generate CSSOM, and then generate the final RenderTree in one go after parsing HTML, rendering only once. If CSS is placed at the bottom of HTML, rendering stutter may occur, affecting performance and experience.

Finally, during the rendering process, if `<script>` is encountered, it will stop rendering and execute the JS code. Because browser rendering and JS execution share a thread, and there must be single-threaded operation here, multithreading can cause rendering DOM conflicts. After the content of `<script>` is executed, the browser continues to render. Finally, consider another question. **Why should JS be placed at the bottom of HTML?** Placing JS at the bottom can ensure that the browser prioritizes the rendering of existing HTML content, allowing users to see the content first, providing a good experience. Additionally, if JS execution involves DOM operations, it must wait for DOM parsing to complete. When JS is executed at the bottom, HTML is definitely parsed into a DOM structure. If JS is placed at the top of HTML, when JS is executed, HTML may not have time to be converted into a DOM structure, which may cause an error.

## Performance Optimization

Performance optimization is a common interview topic. This kind of topic has a lot of extensibility, can expand many small details, and poses a great challenge to personal technical vision and business capabilities. In this part, I will focus on the commonly used performance optimization solutions.

> Topic: Summarize the solutions for front-end performance optimization

### Optimization Principles and Directions

The principle of performance optimization is to use a better user experience as the standard, specifically to achieve the following goals:

- More use of memory, cache or other methods

- Reduce CPU and GPU calculations for faster display

There are two directions for optimization:

- Reduce page size, improve network loading

- Optimize page rendering

### Reduce Page Size, Improve Network Loading

- Compression and merger of static resources (JS code compression and merger, CSS code compression and merger, sprite images)

- Static resource caching (adding MD5 to resource names)

- Use CDN to make resource loading faster

### Optimize Page Rendering

- Put CSS at the front, JS at the back

- Lazy loading (image lazy loading, dropdown for more loading)

- Reduce DOM queries, cache DOM queries

- Reduce DOM operations, try to combine multiple operations to execute (use DocumentFragment)

- Event throttling

- Execute operations as early as possible (DOMContentLoaded)

- Use SSR backend rendering, directly output the data to HTML, reduce the time for the browser to use JS templates to render page HTML

### Detailed Explanation

####Compression and merger of static resources

If not merged, each will go through the request process described before

```
<script src="a.js"></script>
<script src="b.js"></script>
<script src="c.js"></script>
```

If merged, only one request process is walked through

```
<script src="abc.js"></script>
```

#### Static Resource Caching

Control cache through link name

```
<script src="abc_1.js"></script>
```

Only when the content changes will the link name change

```
<script src="abc_2.js"></script>
```

This name doesn’t need to be manually changed, the front-end build tool can add an MD5 suffix to the file name based on the file content.

#### Use CDN to make resource loading faster

The CDN will provide a professional loading optimization solution, and static resources should be placed on the CDN as much as possible. For example:

```
<script src="https://cdn.bootcss.com/zepto/1.0rc1/zepto.min.js"></script>
```

#### Use SSR backend rendering

Directly output HTML content in one time, and do not need to load data and render again through Ajax after the page is completely rendered. For example, using smarty, Vue SSR, etc.

#### CSS comes first, JS comes later

It has been mentioned when describing the browser rendering process above, so I won’t go into details here.

#### Lazy Loading
Initially assign a universal preview image to the src, and then dynamically assign it to the official image when scrolling down. For example, preview.png is a preview image, which is relatively small, loads quickly, and many images share this preview.png, which can be loaded only once. When the page scrolls down and the image is displayed, it is replaced with the value of src as data-realsrc.

```
<img src="preview.png" data-realsrc="abc.png"/>
```

In addition, why should data- be used to start property values here? —— All custom properties in HTML should start with data-, because the browser will ignore properties that start with data- when rendering, which can improve rendering performance.

#### Cache DOM queries

Compare two code blocks:

```
var pList = document.getElementsByTagName('p')  // Query a DOM element once and cache it in pList
var i
for (i = 0; i < pList.length; i++) {
  ...
}
```

javascript

```
var i
for (i = 0; i < document.getElementsByTagName('p').length; i++) {  
  // Each loop queries the DOM, which consumes performance
}
```

In summary: both querying and modifying the DOM are performance-consuming operations, so try to reduce them as much as possible.

#### Merge DOM Insertions

DOM operations are highly performance-consuming, so when inserting multiple tags, first insert the Fragment and then insert into the DOM uniformly.

```
var listNode = document.getElementById('list')
// To insert 10 li tags
var frag = document.createDocumentFragment();
var x, li;
for(x = 0; x < 10; x++) {
    li = document.createElement("li");
    li.innerHTML = "List item " + x;
    frag.appendChild(li);  // First put in frag, then insert into DOM structure at once.
}
listNode.appendChild(frag);
```

#### Event Throttling

For example, you need to trigger a change event when the text changes, which is monitored through keyup. Use throttling.

```
var textarea = document.getElementById('text')
var timeoutId
textarea.addEventListener('keyup', function () {
    if (timeoutId) {
        clearTimeout(timeoutId)
    }
    timeoutId = setTimeout(function () {
        // Trigger change event
    }, 100)
})
```

#### Execute Operations as Early as Possible

```
window.addEventListener('load', function () {
    // All page resources must be loaded before executing, including images, videos, etc.
})
document.addEventListener('DOMContentLoaded', function () {
    // It can be executed once the DOM is rendered, at this time, images and videos may not have been loaded yet.
})
```

#### How to Do Performance Optimization?

The above are individual points of performance optimization. When implementing a performance optimization project, it should be promoted in the following steps:

1. Establish a performance data collection platform, baseline the current performance data, and record the time consumption of the entire page opening process through performance points.

2. Analyze the reasons for the long time period, find the optimization points, and determine the optimization goals.

3. Start optimizing.

4. Record the optimization effect through the data collection platform.

5. Continually adjust optimization points and expected targets, and cycle stages 2 to 4.

Performance optimization is a long-term task, not a one-time success. It should be done gradually based on the principle of base lining first, analyzing carefully, and optimizing last.

## Web Security

> Topic: What are the common security issues in the front end?

There are two major safety concerns in web front-end development. If you can answer the two questions below, you should be able to answer this question. But before beginning, let’s talk about a very simple attack method — SQL Injection.

When I was in school, I knew about an attack method called “SQL Injection”. For example, when making a login interface for a system, entering a username and password, and submitting, the backend directly gets the data and splices the SQL statement to query the database. If malicious SQL assembly is performed at the time of input, the final generated SQL will be problematic. But now, somewhat large systems won’t do this. From submitting login information to finally getting authorization, there has to be multiple layers of validation. Therefore, SQL injections only happen in smaller, lower-end systems.

### XSS (Cross Site Scripting)

This is the most common attack method in the front end, and many large websites (like Facebook) have been attacked by XSS.

For example, I post a normal article on a blogging website, inputting Chinese, English, and pictures, and there’s no problem. But if I write in malicious JS scripts, like getting document.cookie and then transmitting it to my own server, then every time my blog post is viewed, this script will execute and secretly pass the information in the visitor's cookies to my server.

In principle, the hacker inputs a specific JS code covertly in some way (publishing an article, commenting, etc.). Then, when other people read this article or comment, the JS code that was injected before executes. Once the JS code executes, it cannot be controlled, because it has the same permissions as the original JS on the web page, such as getting server-side data, getting cookies, etc. And that’s when the attack happens.

### Harm of XSS

The harm of XSS is quite significant, and if a page can execute other’s unsafe JS code freely, it could cause the page to become chaotic or even lose functionality at a minimum, and at its worst, cause the user’s information to be exposed.

For example, in early years, social networking sites often had XSS worms. Through posting articles with inserted JS scripts, users who visit the infected articles would automatically republish new articles. These articles would enter each user’s article list through the recommendation system, leading to widespread infection.

Another example is using methods to get cookies, transmitting the cookies to the invader’s server. The invader can then simulate the cookies to log in to the website and alter the user’s information.

### Prevention of XSS

So how do we prevent XSS attacks? — — The most fundamental way is to validate and replace the content input by the user. The characters that need to be replaced are:

```
& replaced with: &amp;
< replaced with: &lt;
> replaced with: &gt;
" replaced with: &quot;
' replaced with: &#x27;
/ replaced with: &#x2f;
```

After replacing these characters, the attack code input by the hacker will become invalid, and the XSS attack will not occur easily.

In addition, you can have stronger control over cookies, such as adding http-only restrictions to sensitive cookies, preventing JS from getting the contents of cookies.

### CSRF (Cross-site request forgery)

CSRF borrows the permission of the current operator to secretly complete an operation, and doesn’t take the user’s information.

For example, on a payment website, the interface for transferring money to others is http://buy.com/pay?touid=999&money=100, and there's no password or token verification when using this interface, only simply visiting the webpage will allow for a direct transfer for others. A user has already logged onto http://buy.com. During the selection of goods, he suddenly receives an email, and this email has a line of code `<img src="http://buy.com/pay?touid=999&money=100"/>`. After he visits the email, the purchase has actually been completed.

CSRF actually takes advantage of a feature of cookies. As we know, after logging into http://buy.com, the cookie will have the login marker, and now when you request http://buy.com/pay?touid=999&money=100, it will bring the cookie, which informs the server end that you've logged in. However, if you request another domain's API, such as http://abc.com/api on http://buy.com, it will not bring the cookie, this is restricted by the browser's same-origin policy. But —— now, when you're on another domain's page and you request http://buy.com/pay?touid=999&money=100, it will bring buy.com's cookie. This is the theoretical foundation for a CSRF attack.

The prevention of CSRF is adding permission verification at various levels, such as in current online shopping websites, any transaction involving money will definitely require a password or a fingerprint. Apart from this, sensitive interfaces using POST requests rather than GET is also very important.
