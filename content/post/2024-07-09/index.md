---
title: 'Can sessionStorage Share Data Across Multiple Tabs?'
date: 2024-07-03T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*dEcEJcGtium2C7FJaWrSuA.png"
categories: ["Story","Fiction", "Life"]
tags: ["Story","Fiction", "Life"]
URL: ""
layout: posts
is_recommend: true
description: "`localStorage` and `sessionStorage` are often discussed together, but what are the differences between them?..."
---

`localStorage` and `sessionStorage` are often discussed together, but what are the differences between them?

> The localStorage read-only property of the window interface allows you to access a Storage object for the Document’s origin; the stored data is saved across browser sessions.

> localStorage is similar to sessionStorage, except that while localStorage data has no expiration time, sessionStorage data gets cleared when the page session ends — that is, when the page is closed. (localStorage data for a document loaded in a “private browsing” or “incognito” session is cleared when the last “private” tab is closed.) — [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)

`localStorage` can share data across different tabs under the same website. Here's an example:

```
// You can store data in the first tab
localStorage.setItem('name', 'fatfish');

// And read this data in another tab
localStorage.getItem('name'); // fatfish
```

## What about sessionStorage?

[MDN](https://developer.mozilla.org/en-US/docs/Web/API/Window/sessionStorage) explain:

> - Whenever a document is loaded in a particular tab in the browser, a unique page session gets created and assigned to that particular tab. That page session is valid only for that particular tab.

> - A page session lasts as long as the tab or the browser is open, and survives over page reloads and restores.

> - Opening a page in a new tab or window creates a new session with the value of the top-level browsing context, which differs from how session cookies work.

> - Opening multiple tabs/windows with the same URL creates sessionStorage for each tab/window.

> - Duplicating a tab copies the tab’s sessionStorage into the new tab.

> - Closing a tab/window ends the session and clears objects in sessionStorage.

Based on the second point, you might think that data can be shared across new pages opened from the current page. Try executing the following code in the console of the current page:

```
sessionStorage.setItem('name', 'fatfish');
```

Then, open another page of the same site in a new tab. You won’t be able to read the data:

```
sessionStorage.getItem('name'); // null
```

So, the conclusion is that sessionStorage cannot share data across multiple tabs or windows. Does this mean the explanation on MDN is incorrect?

## The Correct Explanation

sessionStorage indeed cannot share data between multiple windows or tabs. However, when you open a new page via window.open or a link (not a new window), the new page will copy the sessionStorage of the original page.

Here’s an example to illustrate this:

```
sessionStorage.setItem('name', 'fatfish');
window.open('your-page-url.html');

// or

<a href="https://..." />
```

Thus, while sessionStorage does not share data between different windows or tabs, it does copy the data when opening a new page through certain methods.
