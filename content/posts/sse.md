---
title: "What are SSE, Server-Sent Events? Example with Express.js"
date: 2024-09-22
categories:
  - Internet
tags:
  - Node.js
  - Internet
thumbnailImagePosition: left
thumbnailImage: images/javascript.svg
---
Recently I have been using Server-Sent Events (SSE) to develop a feature where the server can notify the client at specific points in time. This articfle will   provide an easy explaintion of what SSE is, when to use it, and how to implement it on both the client and server sides.

## 1. Introduction
Imagine you need real-time updates on your website, such as displaying live stats from the server. While WebSocket might come to mind for creating bidirectional connection, that could be overkill if you only need one-way communication from server to client. This is where Server-Sent Events (SSE) become especially useful.

## 2. What is SSE?
SSE allows the server to push updates to the client over HTTP, without the need for a complex bidirectional connection like WebSocket. Once the connection is established, the server can stream simple text message to the client in real time. Unlike WebSocket, SSE is unidirectional - only the server can send send messages to the client - making it simpler to implement and idea for use case like:
- Progress Updates (e.g., task completion status)
- Time-sensitive Data (e.g., stock price updates)
- Monitoring Dashboards

![SSE](/images/sse.png)

## 3. SSE Message Format
SSE operates over HTTP and uses the **Content-Type: text/event-stream** header. Key message components include:
- **event**: The event name browser will listen for, such as `message`, `error`, or `open`.
- **data**: The data of the return message from server.
- **id**: The last event ID string.
- **retry**: The time in milliseconds before attempting to reconnect.

## 4. Client-side Implimentation
The client opens a persistent connection using an **EventSource** instance, which listens for server-sent events. By default, the `message` event is triggered.

```javascript
const source = new EventSource("http://example.com");

source.onopen = (event) => {
  console.log(event);
};

source.onmessage = (event) => {
  console.log(event);
};

source.onerror = (error) => {
  console.log(error);
	eventSource.close(); // Close the connection to stop retrying
};
```
Modern browsers s.pport SSE through the EventSource API. If your browser doesn't or you need to send custom headers (e.g., JWT authentication), consider using the eventsource-polyfill package:

```javascript
import EventSource from "eventsource";

const eventSource = new EventSource('http://your-path-to-server', {
  headers: {
    'Authorization': `Bearer ${yourJWT}`
  }
});
```
Notice that most browsers support only up to 6 concurrent connections per domain.

## 5. Server-side Eexample with Expres.js
Here's how to implement SSE on the server using Expess.js. The server waits for the client connection and sends a message every 3 seconds. When the client disconnects, the server cleans up the connection.
```javascript
import express from "express";

const app = express();
const PORT = process.env.PORT || 3000;

app.get("/sse", (req, res) => {
  res.setHeader("Content-Type", "text/event-stream");
  res.setHeader("Cache-Control", "no-cache");
  res.setHeader("Connection", "keep-alive");

  // Send initial message
  res.write("data: SSE Connection established.\n\n");

  // Send updates every 3 seconds
  const intervalId = setInterval(() => {
    const message = `Message from server. Timestamp: ${new Date()}`;
    res.write(`data: ${JSON.stringify(message)}\n\n`);
  }, 3000);

  // Hangle client disconnection
  req.on("close", () => {
    clearInterval(intervalId);
    res.end();
    console.log("Client disconnected");
  });
});

app.listen(PORT, () => {
  console.log(`Running SSE at http://localhost:${PORT}/sse`);
});
```
Client-side Output:

```javascript
Onopen: Event { type: 'open' }
Onmessage: MessageEvent {
  type: 'message',
  data: 'SSE Connection established.',
  lastEventId: '',
  origin: 'http://localhost:3000'
}
Onmessage: MessageEvent {
  type: 'message',
  data: '"Message from server. Timestamp: Sat Sep 21 2024 20:49:49 GMT+0800 (Taipei Standard Time)"',
  lastEventId: '',
  origin: 'http://localhost:3000'
}
Onerror: Event { type: 'error', message: undefined } // If server terminate the connection
```

You can check the status of connection using `netstat`.

```bash
$ netstate -an | grep 3000
Active Internet connections
Proto Recv-Q Send-Q  Local Address          Foreign Address        (state)
tcp6       0      0  ::1.3000               ::1.60069              ESTABLISHED
tcp6       0      0  ::1.60069              ::1.3000               ESTABLISHED
tcp46      0      0  *.3000                 *.*                    LISTEN
```

## Conclusion
Server-Sent Events provide a simple, unidirectional method for streaming real-time updates from the server to the client over HTTP. It’s easier to implement compared to WebSockets, especially for scenarios where the server only needs to push updates.

---

## References

- [MessageEvent interface in w3c](https://html.spec.whatwg.org/multipage/comms.html#the-messageevent-interface)
- [SSE in w3c](https://html.spec.whatwg.org/multipage/server-sent-events.html)
- [MDN Server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events)
- [EventSource MDN](https://developer.mozilla.org/en-US/docs/Web/API/EventSource)
- https://www.pubnub.com/guides/server-sent-events/
