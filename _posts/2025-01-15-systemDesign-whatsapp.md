---
layout: post
title: "System design - whatsapp (chat app)"
date: 2025-01-15
description: "System design"
tag: System Design
---

# Chat App

## [digest](https://dev.to/karanpratapsingh/system-design-whatsapp-fld)

Basic functional requirements

1. one-on-one chat

2. group chat

3. media

add-on feature

1. sent, dilevered, read receiptent

2. push notification

3. last seen time

## Traffic & Storage

### Traffic

say 50 million DAU, each user sent 10 messages to 4 different users on average everyday. Then we have 50 million \* 40 = 2 Billion message / day.

assume 5% of messages are media files -> 100 million media file

### Storage

assume each message is 100 bytes on average. 2 billion \* 100 bytes = 200 GB/day

for media file, assume 50k for each on average, 2 billion _ 5% _ 50k = 5TB/day

each day will be 5.2TB, that is 5.2TB _ 365 _ 5 = 9.5 PB

### BandWidth

The bandwidth for message is 2 billion \* 100 bytes / 86400 sec = 23 MB / s

bandwidth for media file is 5 TB / 86400 s = 58 M/s

## Api design

Start with the whatsapp home screen, there will be a list of history chat or gourp you are in, so we need to get all history chat or gourp a user in.

1. get all chats & groups of user

```
getAllChats(userId) --> chats[] | groups[]
Parameters:
    userId: current user
return:
    list of chats or groups
```

For each chat or gourp chat, need to have history message

2. get chat history message

```
getMessages(channelId) --> message[]
parameters:
    channelId: the channelId want to retrieve
return:
    list of message
```

3. send message

```
sendMessage(userId, channelId, message)
parameter:
    userId: current user
    channelId: the channel user want to send the message to
    message: object contains content, type ...
return:
    boolean: success or not
```

4. join or leave chat

```
joinGroup(userId, channelId)
leaveGroup(userId, channelId)
return:
    boolean: success or not
```

## High Level Design

### Modeling (Optional)

<img src="https://miro.medium.com/v2/resize:fit:720/format:webp/0*8AhupYvEef-A4Gk3.png">

We have the following tables:

**users**

This table will contain a user's information such as name, phoneNumber, and other details.

**messages**

As the name suggests, this table will store messages with properties such as type (text, image, video, etc.), content, and timestamps for message delivery. The message will also have a corresponding chatID or groupID.

**chats**

This table basically represents a private chat between two users and can contain multiple messages.

**users_chats**

This table maps users and chats as multiple users can have multiple chats (N:M relationship) and vice versa.

**groups**

This table represents a group between multiple users.

**users_groups**

This table maps users and groups as multiple users can be a part of multiple groups (N:M relationship) and vice versa.

### feature implementation

1. Real-time messaging

Pull model: Client periodically send HTTP request to servers to get updates. - long polling

push model: client opens a long-lived connection with server and once new data is available, it will be pushed to the client. - WebSocket - Server-Sent Events(SSE)

pull model will create unnecessary request overhead on servers as most of the time the response is empty.

To minimize latency, using push model with websocket is a better choice.

2. Last seen

Use heartbeat mechanism. client periodically ping the server to tell it is active. When lose the signal, mark the client is offline.

Apply cache on it, like user - last seen timestamp - Redis - memcached

**Use Redis for real-time presence tracking (with a heartbeat mechanism). Redis can handle millions of real-time updates per second.**

**Instead of constantly querying for the presence, introduce a status expiration mechanism where users are marked as offline after a certain threshold if they haven’t sent updates for a while (e.g., 60 seconds). This reduces constant writes to the Redis store**

(or we can use (lazy update approach) last activity crosses a certain threshold like user has not performed any action in last 30 seconds)

3. Read receipts

Handling read receipts can be tricky, for this use case we can wait for some sort of Acknowledgment (ACK) from the client to determine if the message was delivered and update the corresponding deliveredAt field. Similarly, we will mark message the message seen once the user opens the chat and update the corresponding seenAt timestamp field.

4. Push Notification

once message is sent in chat, check if recipient is active or not. If not actice, add an event to message queue with metadata like client device platform which will be used to route the notification.

then consume message, forward request to **FCM** (Firebase Cloud Messaging) or **APNS** (apple Push notification service)

---

Now focus on the high level design

<img src="https://media2.dev.to/dynamic/image/width=800%2Cheight=%2Cfit=scale-down%2Cgravity=auto%2Cformat=auto/https%3A%2F%2Fdev-to-uploads.s3.amazonaws.com%2Fuploads%2Farticles%2Fibq8zcctgrz760s1smf6.png">

**User Service**

This is an HTTP-based service that handles user-related concerns such as authentication and user information.

**Chat Service**

The chat service will use WebSockets and establish connections with the client to handle chat and group message-related functionality. We can also use cache to keep track of all the active connections sort of like sessions which will help us determine if the user is online or not.

**Notification Service**

This service will simply send push notifications to the users.

**Presence Service**

The presence service will keep track of the last seen status of all users.

**Media service**

This service will handle the media (images, videos, files, etc.) uploads.

---

For partitioning, we can use consistant-hashing.

For **cache**, be careful to use as user want to see latest message. We can cache some older message to prevent usage spike (peek hours).

use Redis or memcached with LRU cache eviction policy (discard least recently used).

To improve system, use **pagination** when fecting messages in group chat. retrieve old message only when request.

For media storage, use AWS S3 or google cloud storage. Use CDN to provide geo cache to speed up the process.

**API Gateway** can offer features like authentication, authorization, rate limiting, throttling.

## Bottleneck identify and solution

1. Real-time messaging

A websocket connection per user is required to be maintained, should be managed properly. Add load-balancer ahead of chat service to reduce the load on each chat service.

2. message storage

When retrieve chat history, for chats that has too many messages. use **lazy-load** and **pagination** to improve reduce the load.

3. Write heave throughput

use Nosql - cassandra to store message as it is a write heavy work.

offload media traffic using CDN.

use **chunk upload** to offer better reliability.
