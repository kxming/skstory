---
title: 'How to refresh token seamlessly'
date: 2024-06-07T16:02:05+08:00
draft: false
author: ""
image: "https://miro.medium.com/v2/resize:fit:1400/format:webp/1*nENIDWcdJkPzfhte4UmYNQ.png"
categories: ["Code","Javascript"]
tags: ["Code","Javascript"]
URL: ""
layout: posts
is_recommend: true
description: "In front-end development, implementing seamless token refresh is usually to maintain the user’s login status and ensure the validity of their access permissions."
---

In front-end development, implementing seamless token refresh is usually to maintain the user’s login status and ensure the validity of their access permissions.

## Implementation principle
The implementation of seamless token refresh typically involves the following steps:

- **Token Expiration Time**: Initially, the server assigns an expiration time to each token, such as 30 minutes or 1 hour. During this period, users can use the token to access protected resources.

- **Regular Checks**: The front-end application periodically checks the token’s expiration during user activity. This is usually accomplished through polling or a heartbeat mechanism, which involves sending a request to the server at set intervals to check if the token is still valid.

- **Refresh Token**: If the server indicates the token is expired, the front-end application will immediately make a new request to the authentication server using a refreshToken (typically, the backend will provide a long-term refreshToken and a short-term accessToken, where ‘long’ and ‘short’ refer to the expiration time; the refreshToken is stored in local storage to request a new accessToken from the backend, and the accessToken is used in the request headers to access protected endpoints) to obtain a new access token.

- **Seamless Transition**: After obtaining the new token, the front-end application will update the token stored locally and continue the previous actions with the new token, thereby providing a seamless user experience.

### Example:

Here is a simple example showing how to implement seamless token refresh in a front-end application:

- **Set Token Expiration Time**: Assume the server sets a 30-minute expiration time for each token.

- **Regular Checks**: The front-end application sends a request to the server every 5 minutes to check the current token’s validity.

```
// Use setInterval to send heartbeat requests periodically
setInterval(() => {
  checkTokenValidity();
}, 5 * 60 * 1000); // every 5 minutes
```

- **Check Token Validity**: The front-end application sends a heartbeat request to the server, which returns information about whether the token is valid or not.

```
async function checkTokenValidity() {
  try {
    const response = await fetch('/api/heartbeat', {
      headers: {
        Authorization: `Bearer ${localStorage.getItem('token')}`
      }
    });
    if (!response.ok) {
      // Token expired, refresh needed
      refreshToken();
    }
  } catch (error) {
    // Handle errors
    console.error('Error checking token validity:', error);
  }
}
```

- **Refresh Token**: If the token has expired, the front-end application makes a request to the authentication server to obtain a new access token.

```
async function refreshToken() {
  try {
    const response = await fetch('/api/auth/refresh', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        refreshToken: localStorage.getItem('refreshToken')
      })
    });
    if (response.ok) {
      const data = await response.json();
      // Update the tokens in local storage
      localStorage.setItem('accessToken', data.accessToken);
    } else {
      // Handle failed token refresh, for example, prompt the user to log in again
      console.error('Failed to refresh token:', response.status);
    }
  } catch (error) {
    // Handle errors
    console.error('Error refreshing token:', error);
  }
}
```

In this example, the front-end application maintains user login status and provides a seamless user experience by regularly checking token validity and seamlessly refreshing the token upon expiration. For security reasons, it’s important to note that the refreshToken should also have an expiration time, after which users are required to log in again.

---

## How to handle other requests sent while the token is still being refreshed?

If the token refresh isn’t complete yet and you’ve sent other requests with the old token to the server, you might encounter the following situations:

**Request Denied**: If the server detects that the token has expired, it may return a 401 Unauthorized or a similar error status code. In this case, you need to catch these errors and try to resend these requests with the new token after catching the errors.

To handle these situations, you can adopt the following strategies:

1. **Error Handling**: Ensure your application can catch and handle 401 Unauthorized or other related errors. When these errors are caught, you can try to resend the requests with the new token.

2. **Retry Mechanism**: Implement a request retry mechanism that allows for automatic or manual retries with a new token when a request fails. You can use an exponential backoff strategy to avoid putting pressure on the server with frequent retries.

3. **State Management**: During token refresh, you can set the application state to “refreshing token” and prevent sending new requests until a new token is obtained. This avoids sending more requests with an invalid token.

4. **Queue Requests**: Instead of immediately sending requests after the token expires, you can queue them and send them sequentially once the new token is obtained. This can be accomplished using Promise.all or other asynchronous processing techniques.

5. **Front-End Notification**: After the token expires, you can display a notification on the front end informing the user that the token is being refreshed and that they may need to wait a while or login again.

### Example:

This example shows how to resend requests using a new token after capturing a 401 error:

```
// Assume this is your API request function
async function apiRequest(url, token) {
  try {
    const response = await fetch(url, {
      headers: {
        Authorization: `Bearer ${token}`
      }
    });
    if (!response.ok) {
      throw new Error(`Request failed with status ${response.status}`);
    }
    return response.json();
  } catch (error) {
    if (error.message.includes('401') && newToken) {
      // Try resending the request with the new token
      return apiRequest(url, newToken);
    }
    throw error;
  }
}
// Assume this is your token refresh function
let newToken = null; // Variable to store the new token
async function refreshToken() {
  try {
    const response = await fetch('/api/auth/refresh', {
      // ... code for sending refresh request ...
    });
    if (response.ok) {
      const data = await response.json();
      newToken = data.token;
    }
  } catch (error) {
    // Handle error
  }
}
// Call this function when the token expires
async function handleTokenExpiration() {
  try {
    // Try refreshing the token
    await refreshToken();
    // Assume this is the array of failed requests due to token expiration
    const failedRequests = [/* ... */];
    // Resend requests using the new token
    for (const request of failedRequests) {
      try {
        const response = await apiRequest(request.url, newToken);
        // Process response or update application state
      } catch (error) {
        // Handle errors while resending requests
      }
    }
  } catch (error) {
    // Handle errors while refreshing the token or resending requests
  }
}
```

In this example, the apiRequest function checks if there is a new token available when encountering a 401 error and tries to resend the request using the new token. The handleTokenExpiration function is responsible for refreshing the token when it expires and resending previously failed requests.

### Queueing Requests

The implementation of queued requests primarily utilizes a queue data structure to manage pending requests. When a request needs to be delayed for some reason (such as a token expiration), we can place this request in a queue and wait for conditions to be met (like receiving a new token) before processing the request from the queue.

Below is a simple example to illustrate implementing queued requests:

**1. Create a Request Queue**

First, we need a queue to store pending requests. This queue can be an array or a linked list in memory, or it could use existing queue libraries (such as JavaScript’s `Array.prototype.queue` or the `queue-promise` library).

```
let requestQueue = []; // Use an array as a simple queue implementation
```

**2. Enqueue Operation**

When we need to delay processing a request, we add it to the end of the queue.

```
function enqueueRequest(request) {
  requestQueue.push(request);
}
```

**3. Dequeue Operation**

Once conditions are met (like a new token being available), we process a request from the head of the queue and repeat this process until the queue is empty.

```
async function dequeueAndProcessRequests() {
  while (requestQueue.length > 0) {
    const request = requestQueue.shift(); // Take a request from the head of the queue
    try {
      const response = await processRequest(request); // Process the request
      // Process response or update application state
    } catch (error) {
      // Handle errors during request processing
      console.error('Error processing request:', error);
    }
  }
}
```

**4. Process Requests**

The `processRequest` function is responsible for the actual request processing logic. In this function, you can send HTTP requests, update the UI, etc.

```
async function processRequest(request) {
  // Here is the logic for processing the request, like sending an HTTP request
  const response = await fetch(request.url, {
    headers: {
      Authorization: `Bearer ${newToken}` // Use the new token
    }
  });
  return response.json(); // Return the processing result
}
```

**5. Usage Example**

When the token expires, we can queue requests that need to be delayed for processing, and then try to refresh the token. Once we get the new token, we process the requests from the queue.

```
// Suppose this is a request that needs to be delayed due to token expiration
const delayedRequest = { url: '/api/data' };
// Add the request to the queue
enqueueRequest(delayedRequest);
// Try refreshing the token
await refreshToken();
// Process the requests in the queue
await dequeueAndProcessRequests();
```

### Considerations

- **Queue Size Management**: If the request queue becomes very large, it might consume a lot of memory. You might need to implement strategies to limit the queue size, such as setting a maximum length or discarding the oldest requests.

- **Error Handling**: Ensure proper error handling when processing requests and responses to avoid application crashes or unpredictable behavior.

- **Concurrency Control**: If your application needs to handle a large number of concurrent requests, you might need mechanisms to control the concurrency, such as using Promise.all to limit the number of requests processed simultaneously.
