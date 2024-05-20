---
title: 'Why does WebSocket require frontend heartbeat detection, and is there a native detection mechanism?'
date: 2024-05-20T16:22:38+08:00
draft: false
author: ""
image: "https://cdn-images-1.medium.com/max/1600/1*K6zrMqjRYIpYEB6reDP_yA.jpeg"
categories: ["Code","low-code"]
tags: ["Code","low-code"]
URL: ""
layout: posts
is_recommend: true
description: "In the software industry, there is a famous book called “The Mythical Man-Month,” which includes a discussion that there is “no silver bullet.”"
---

In web applications, WebSocket is a very common technology. A WebSocket connection can be established simply by using the WebSocket constructor in the browser. However, when it's applied to specific projects, heartbeat detection is almost always implemented.

The purpose of setting up heartbeat detection is twofold: first, to confirm that both parties in the communication are still active, and second, to allow the browser side to promptly check the availability of the current network connection, ensuring the timeliness of message delivery.

You might think, is WebSocket so basic that it can't determine its connection status? Before understanding this further, let's first review some computer network knowledge.

## Related Network Knowledge

The four-layer structure of the TCP/IP protocol suite:

- Application Layer: Determines the activities of communication when providing application services to users. Protocols like HTTP, FTP, and WebSocket operate on this layer.

- (Transport Control Protocol) TCP layer: Controls the data transmission between two hosts in the network by segmenting application layer data (for example, segmenting a complete HTTP message) and sending it to the application program at a specific port on the target host. It tags each data segment with a source port, target port, and sequence number after segmentation.

- (Internet Protocol) IP layer: Maps IP addresses to MAC addresses of target hosts, then adds information like source IP, and target IP to the TCP packets (data fragmentation if necessary) and sends them over the network through the link layer to find the target host.

- Link Layer: Sends and receives datagrams for the IP network layer, converting binary data packets to and from network electrical signals transmitted over Ethernet cables.

> TCP is a reliable connection-oriented protocol. After establishing a handshake connection, the sender expects a confirmation message from the receiver for each TCP message sent (multiple TCP messages may be formed after application layer message segmentation) within a specified time. If there is no response, it will resend, ensuring all TCP messages reach the receiver and are reassembled in order by the receiver into the complete message needed at the application layer.

> The WebSocket protocol supports introducing a TLS layer on top of TCP to establish encrypted communication.

**The differences and similarities between WebSocket and HTTP:**

- Like HTTP, WebSocket is an application layer protocol that uses TCP at the transport layer and is a reliable connection. When establishing a WebSocket connection, it can use an existing HTTP GET request to handshake: the client sends information such as the WebSocket protocol version in the request headers to the server, and if the server agrees, it will respond with a status code of 101. In other words, a single HTTP request and response can smoothly switch the protocol to WebSocket.

- WebSocket allows for bidirectional requests. When there is a new message, the server can actively notify the client without the client having to inquire actively. Clients can also send messages to the backend. In HTTP, requests can only be initiated by the client.

- WebSocket is part of HTML5, while HTTP is the Hypertext Transfer Protocol, which predates the birth of HTML5.
At the application layer, each WebSocket message (called a data frame in WebSocket) is lighter than an HTTP message (which must include a request line, headers, and data).

WebSocket data frames contain only fixed, lightweight header information without cookies or custom headers. After establishing communication, it is a one-to-one interaction and does not require the transmission of authentication information. However, the handshake HTTP request will automatically carry cookies.
WebSocket at the application layer will split large data into multiple frames, while HTTP does not split each message.

**The differences and similarities between WebSocket and WebRTC:**

WebRTC is a communication technology initiated by Google and implemented by a wide range of browsers. It is used to establish communication between browsers, such as video calls. Whereas WebSocket is a protocol abstraction that can be implemented as a communication technology to establish communication between browsers and servers.

## Mechanism of Heartbeat Detection in Protocols

From the answers found online, there are approximately two ways from the perspective of protocols for WebSocket to detect if the other party is alive:

1. WebSocket is merely a specification for an application layer protocol, with its transport layer being TCP, which provides a KeepAlive mechanism for long connections. This mechanism allows for the periodic sending of heartbeat messages to confirm the survival of the other party, but it's generally used by the server side. Since it's a mechanism of the TCP transport control layer, its specific implementation depends on the operating system. In other words, the connection status received at the application layer is notified by the operating system. Different operating systems schedule resources differently, such as when to send probe messages (TCP messages without valid data) to detect the survival of the other party, and how frequently, with the frequency varying under different system configurations. It might be once every two hours, or possibly shorter. Only when no response packets are received consecutively will the application layer be notified of the disconnect. This introduces uncertainty. It also means that other application layer protocols relying on this mechanism will be affected. That is to say, to utilize this process for detection, the client side will have to modify the TCP configuration of the operating system, which doesn't work in a browser environment.

2. The WebSocket protocol also has its keep-alive mechanism, but it requires implementation by both communicating parties. Data frames in WebSocket communication have a 4-bit OPCODE that marks the type of the current data frame, for example, 0x8 for the close frame, 0x9 for the ping frame, 0xA for the pong frame, 0x1 for the ordinary text data frame, [etc.www.rfc-editor.org](https://etc.www.rfc-editor.org/)

- Close data frame: When either party wants to close the channel, it sends a data frame to the other party with the OPCODE for connection closing. For example, when the WebSocket instance in the browser calls close, it sends a data frame with the OPCODE indicating a connection close to the server side, which after receiving also needs to return a closing data frame, then close the underlying TCP connection.

- Ping data frame: Serves for the sender to inquire if the other party is still alive, i.e., a heartbeat check package. Currently, only the backend can control the sending of ping data frames. However, there are no corresponding APIs available on the browser side's WebSocket instances.

- Ping data frame: When one party in the WebSocket communication receives a ping data frame sent by the other, it needs to promptly reply with a data frame of consistent content and marked as pong in the OPCODE, to tell the other party it's still present. However, replying to pong is an automatic behavior of the browser, meaning it varies across different browsers. Moreover, there are no APIs available in JS to control this.

To sum up, the methods of detecting the survival of the other party are all initiated by the server performing the heartbeat detection. The browser does not provide such capabilities. To be able to detect the survival of the backend in real-time on the browser end, or in other words, to ensure the connection remains available, one must implement the heartbeat detection themselves.

## The Necessity of Heartbeat Detection on the Browser Side

First, let's understand the circumstances under which a browser-side WebSocket will automatically close and trigger the close event:

- The WebSocket address is not available during the handshake.

- Other unknown errors.

- Under a normal connection, receiving a close frame from the server side will trigger the close callback.

This means that after establishing a normal connection, if the browser side loses the network connection, or if the server closes the connection without sending a close frame - in short, when the connection can no longer be used and the browser has not received a close frame - it will maintain the connection status for a long time. In such a case, if the business code does not actively probe, it will not be able to detect this issue.
Furthermore, both parties maintaining a connection means that resources must be occupied for an extended period. For the server side, resources are extremely precious. Connections that have been inactive for a long time may be "optimized" and released by the server-side application layer framework.

## Frontend Implementation of Heartbeat Detection

Instantiate a WebSocket:

```
function connectWS() {
    const WS = new WebSocket("ws://127.0.0.1:7070/ws/?name=greaclar");
    // Events on the WebSocket instance
    // When the connection is successfully opened
    WS.addEventListener('open', () => {
        console.log('WebSocket connection successful');
    });
    // Listen for messages pushed from the backend
    WS.addEventListener('message', (event) => {
        console.log('WebSocket received a message', event.data);
    });
    // Listen for close messages from the backend, this will also be triggered in case of unexpected errors
    WS.addEventListener('close', () => {
        console.log('WebSocket connection closed');
    });
    // Listen for unexpected error messages on the WebSocket
    WS.addEventListener('error', (error) => {
        console.log('WebSocket error', error);
    });
    return WS;
}
let WS = connectWS();
```

This code snippet demonstrates the basic setup for establishing a WebSocket connection in a front-end environment. The WebSocket is instantiated using the new WebSocket constructor with a specified address, and event listeners are added to handle the connection's open, message, close, and error events.

**Instance methods needed for heartbeat detection:**

```
// Send a message, used for sending a heartbeat package
WS.send('hello'); 
// Close the connection, when the heartbeat package gets no response and a reconnection is needed, it's best to close it first
WS.close();
```

These code lines are crucial for implementing a heartbeat mechanism in a WebSocket connection. By using WS.send('hello');, a simple message or heartbeat package is sent to check the connectivity. If there's no response, indicating potential issues with the connection, WS.close(); is used to properly close the connection before attempting a reconnection.

## Defining the Logic for Sending Heartbeat Packages:

**Preparation**

1. Declare a variable heartbeatStatus to track the current heartbeat check status, with three states: waiting, response received, and timeout.

2. Listen to the message event of the WS instance. Change heartbeatStatus to the response received upon detection.

3. Listen to the open event of the WS instance. Start heartbeat detection after opening.

**Detection**

1. Initiate a Timer A.

2. Upon Timer A’s execution, do the following: a. Change the current state heartbeatStatus to waiting; b. Send the heartbeat package; c. Start another Timer B.

3. After sending the heartbeat package, the backend should immediately push a heartbeat response package with the same content to the frontend, triggering the message event of the frontend WS instance, thereby changing heartbeatStatus to the response received.

4. When Timer B executes, check the current heartbeatStatus state:

  - If its response is received, it means the server can respond to data promptly after Timer A’s execution. Continue by restarting Timer A, and then loop continuously.

  - If it’s waiting, it indicates there are issues with the connection. Proceed to close or go through the detection process.

```
let WS = connectWS();
let heartbeatStatus = 'waiting';
WS.addEventListener('open', () => {
    // Start heartbeat detection after the connection is successfully established
    startHeartbeat()
})
WS.addEventListener('message', (event) => {
    const { data } = event;
    console.log('Received heartbeat response, status should now be set to received', data);
    if (data === '"heartbeat"') {
        heartbeatStatus = 'received';
    }
})
function startHeartbeat() {
    setTimeout(() => {
        // Set status to 'waiting' for a response, and send a heartbeat
        heartbeatStatus = 'waiting';
        WS.send('heartbeat');
        // Start a timer to check if the server has responded
        waitHeartbeat();
    }, 1500)
}
function waitHeartbeat() {
    setTimeout(() => {
        console.log('Checking if the server has responded to the heartbeat, current status:', heartbeatStatus);
        if (heartbeatStatus === 'waiting') {
            // Heartbeat response has timed out
            WS.close();
        } else {
            // Initiate the next round of heartbeat detection
            startHeartbeat();
        }
    }, 1500)
}
```

### Optimizing Heartbeat Detection

When the heartbeat detection fails and the close event is not triggered, there is likely a poor network connection between both parties. Immediately attempting to reconnect could consume more network resources, increasing the chances of a failed reconnection and possibly blocking other user actions.

However, it cannot be ruled out that there is indeed a problem with the connection, such as a server crash, an accidental restart, and the browser not being informed that the old connection needs to be closed.

Therefore, when a heartbeat goes unanswered, I recommend delaying for a while and then informing users that a network issue is being fixed, to prepare them psychologically. Subsequently, send one or two more heartbeat packets; if there is still no response, inform the user of disconnection and ask whether to reconnect. If normality resumes in the meantime, there is no need to reconnect, which would provide a better user experience and put less pressure on the server.

```
// Modifications needed in the code above

// Add a variable to record the number of consecutive no-responses
let retryCount = 0;

WS.addEventListener('message', (event) => {
    const { data } = event;
    console.log('Received heartbeat response, changing status to received', data);
    if (data === '"heartbeat"') {
        // Reset the count of consecutive no-responses
        retryCount = 0;
        heartbeatStatus = 'received';
    }
})
// Add retry logic in the function waiting for responses
function waitHeartbeat() {
    setTimeout(() => {
        // If heartbeat response is normal, start the next round of heartbeat detection
        if (heartbeatStatus === 'received') {
            return startHeartbeat();
        }
        // Update the timeout count
        retryCount++;
        // If the heartbeat response times out but doesn't consecutively exceed three times
        if (retryCount < 3) {
            alert('WS connection is abnormal, checking in progress.')
            return startHeartbeat();
        }
        
        // If the timeout count exceeds three times
        WS.close();
    }, 1500)
}
```
