---
layout: post
title: "Network - RPC, HTTP, gRPC"
date: 2023-09-22
description: "What is RPC, gPRC?"
tag: Computer Network
---

# RPC (Remote Procedure Call)

## What is RPC?

`RPC` is a protocol used to call other processes on the `remote systems` like a `local system`. (remote invocation) Beside HTTP, this is another way to achieve communication in distributed systems.

### Transport protocol and serialize protocol

RPC is a protocol, so it needs a transport protocol and a serialize protocol.

- Transport protocol:
  1.  Dubbo - TCP
  2.  gRPC - HTTP/2
- Serialize protocol: JSON, XML, Protocol Buffer

## RPC VS HTTP

Compared to HTTP, RPC is more efficient and faster. It can `customize` the `protocol` and serialize protocol, which can reduce the size of the data and improve the efficiency of the transmission.

TCP is redundant, the headers are too large, so we can use HTTP/2 to reduce the size of the headers.
