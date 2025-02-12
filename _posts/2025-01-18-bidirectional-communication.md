---
layout: post
title: "System design basic - HTTP Request vs HTTP Long-Polling vs WebSocket vs Server-Sent Events"
date: 2025-01-18
description: "System design"
tag: System Design
---

HTTP Request vs HTTP Long-Polling vs WebSocket vs Server-Sent Events

In this blog, we are going to learn the **HTTP Request** vs **Http Long-Polling** vs **WebSocket** vs **Server-Sent Events(SSE)**.

These are important when it comes to system design interviews.

First, let's have a quick introduction to the HTTP request.

## [](#http-request)HTTP request

Consider there is a client and a server. The client makes a request to the server for the data, the handshaking is done, and the connection gets opened between the client and the server.

Then, the server does its work and sends the response back to the client using that already opened connection. And then finally the connection gets closed. That's it.

One of the use-cases of the HTTP requests is: Fetching the profile information in Facebook applications.

This was a quick introduction to the HTTP request. Now, we need to discuss the HTTP Long Polling.

But, before discussing HTTP Long Polling, we also need to know about HTTP Polling which is actually different from HTTP Long Polling.

## [](#http-polling)HTTP Polling

Once again consider there is a client and a server. The client can make a request to the server for the data using the HTTP request.

So, here the client keeps making the request at a regular interval for example 1–2 seconds whatever it can be.

The server does its work and sends the response back to the client.

Here, we need to notice that the response can be empty because the server might not have any updates which are useful for the client. In that case, most of the requests might get an empty response. And in a few of the requests, the client gets the updates that are useful. So, the problem is that we are making so many unnecessary network calls.

Let's take a look at the WhatsApp example for chatting use-case, we can consider client as the WhatsApp Android or iOS App, the server is the WhatsApp server.

Here the client will be making the request at regular intervals for example 2 seconds. There might be a case where we do not have any new messages, and we will keep polling, and draining the battery of the Mobile device.

But, for sending the message from the client to the server, we can make the HTTP request. But, this is not a good idea when it comes to getting the new messages from the server, because of two reasons:

- First, delayed messages by 2 seconds in many cases as we will poll the server at regular intervals of 2 seconds.
- Second, most of the time, we will be getting empty responses.

We need a better solution than this.

But when we consider another example, like a location update of a delivery boy coming to deliver the food, we are good with 2 seconds delay in the location update. So, HTTP polling can be used. I am not saying it is recommended but can be used for these types of use-cases.

So, this was HTTP Polling. Now, let's learn and try another solution which is HTTP Long Polling.

## [](#http-longpolling)HTTP Long Polling

Once again consider there is a client and a server. Again, the client can make a request to the server for the data using the HTTP request.

But, here is the catch, the client waits for the server to provide the response. There will be a connection opened till the server has a response to send back. As the connection is open for a long time, that's why we call it HTTP Long-Polling. This is why it is different from HTTP Polling.

As soon the server has the response, it sends it back to the client and the connection gets closed. And the client makes another request and waits for the response and this keeps on going in this way again and again.

One more thing to notice is that we also add a timeout to each request. So, when the client either gets the response or a timeout occurs, the connection gets closed, it makes a new request, and starts waiting for the next response.

If we consider our WhatsApp example, this Long HTTP-Polling is a better solution than the HTTP-Polling because

- The message can come in real-time.
- And there will be no empty response although we will have timeout and reconnection again and again.

But again, we can have a better solution than this too.

And this was all about HTTP Long-Polling. Now, we know how it differs from HTTP Polling.

Now, let's learn about WebSocket.

## [](#websocket)WebSocket

First, we need to know about the WebSocket.

From the official documentation:

> A WebSocket is a persistent connection between a client and a server. WebSockets provide a bidirectional, full-duplex communications channel that operates over HTTP through a single TCP/IP socket connection. At its core, the WebSocket protocol facilitates message passing between a client and server.

Let's see how it works.

Again consider there is a client and a server.

First, the client does the WebSocket handshaking with the server, then, the TCP connection gets established between the server and the client through the WebSocket.

Now the important thing to notice here is that, as this is a bi-directional communication, the server can send data to the client anytime, and the client can send data to the server anytime.

Also notice that this will reduce the overhead of handshaking again and again, as we are doing the handshaking only once at the beginning. This way it reduces the overhead.

And this WebSocket has made our task very easy.

Now, when we use the WebSocket for the WhatsApp chat. Things become better, now we can easily send and receive messages. And this is the best solution. And, WhatsApp uses WebSocket.

Now, let's discuss the last one Server-Send Events(SSE).

## [](#server-send-eventssse)Server-Send Events(SSE)

As the name itself tells here server sends events.

According to the definition, it is a server push technology enabling a client to receive automatic updates from a server.

Again consider there is a client and a server. Using SSE, the clients make a persistent long-term connection with the server. Then, the server uses this connection to send the data to the client.

But the client can't send the data to the server using the SSE. This is very important.

For that, we will have to use the normal HTTP request.

So if we see the WhatsApp example again, for getting the message from the server, it will work, but for sending the message, we will have to use the HTTP again, so this is not a better solution. The best solution was WebSocket for the WhatsApp chat use case.

But if we take the example of a real-time stock price application, this SSE can be used as we will get the updates from the server continuously and we will have nothing as such that we will send to the server. So, SSE is a good fit for this use case.

And now we know how HTTP Request, HTTP Long-Polling, WebSocket, and Server-Sent Events differ from each other.

Watch the video format: [HTTP Request vs HTTP Long-Polling vs WebSocket vs Server-Sent Events](https://www.youtube.com/watch?v=8ksWRX4xV-s)

That's it for now.
