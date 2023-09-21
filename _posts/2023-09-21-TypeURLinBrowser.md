---
layout: post
title: "Network - What happend when you type a URL in browser"
date: 2023-09-21
description: "What happend when you type a URL in browser"
tag: Computer Network
---

# What happend when you type a URL in browser

1. Check url is valid or not

2. Check browser cache, system cache, router cache, ISP cache, if cache hit, return the page, else go to step 3

3. DNS resolve, get IP address

4. Browser send HTTP request to server (TCP three-way handshake)

5. After handshake, browser send HTTP request to server

6. Server handle the request, and send back HTTP response

7. Browser receive the response, and render the page to user
