---
layout: post
title: "System design - Url Shorterner"
date: 2025-04-15
description: "Url Shorterner"
tag: System Design
---

## clairify requirements & back-of-the-envelope estimation

url shortern basiclly need to handle 2 flow:

- user give a long url, return a short url
- user type a short url, redirect to original url

Will cover this later, focus on algorithm & deep dive first

## high level design

`API DESIGn`

`URL Redirect`

Redirect can be achieved easily by return the response with HTTP Status code 301 or 302

- `301`: the requested URL is permanently moved to long URL, browser will cache the response and for the following requests with same url will directly go to the long url.

```text
Pros:
    1. reduce the server load by reducing the traffic
Cons:
    2. no available data for analysis & metrics
```

- `302`: the requested URL is temporarily moved to long URL, the request goes to URL shorterning service first then redirect to long URL server.

opposite pros and cons compared with 301

`URL shorterning`

Basiclly store a (shortern URL, long URL) in database, mimic the hashtable. Store in database because memory do not have enough space to store all pair.

`Hash function`

It need a effective hash function to conver long url to short url.

- Each longURL must be hashed to one hashValue.
- Each hashValue can be mapped back to the longURL.

## Deep Dive

`Data model`: since data is well structured and url shortern service is generally read heavy, choose relation databse here. The table will be like:

| id | shortURL | longURL |

`estimate hashvalue length`

in back-of-envelope estimation, assume there will be up to 365 billion URLs. The hashValue consists of characters from [0-9, a-z, A-Z],  containing 10 + 26 + 26 = 62 possible characters.

So 62^n > 365 billion, n = 7. the minimum length of hashvalue is 7.

two approached introduced below:

`Hash + collision resolution`

common hash function are `CRC32`, `MD5`, `SHA-1`. We can choose any of them and choose the first 7 digit of the hashvalue, but this will increase the collision rate.

To prevent collision, if the first 7 digit already exists in our DB, we can add a predefined or random string to the end of the url, then hash again, repeat this until there is no collision.

```text
example:
    1. suppose "apple" hashvalue first 7 digit is "a8d28dh" and already stored in our DB (a8d28dh, apple)
    2.  suppose "orange" hashvalue first 7 digit is also "a8d28dh", so we add a predefined string like ABC at the end of orange. it becomes "orangeABC" and the hashvalue is "t2ad3gh", so we store (t2ad3gh, orange)
```

to quickly tell whether the shortern url is already exists in DB, then we can use `bloom filter` to reduce the cost here.

`Base 62`

Another approch is generate an unique ID for the long url and use Base 62 to reduce the ID.

```text
11157(10) = 2 x 62^2 + 55 x 62^1 + 59 x 62^0 = [2, 55, 59] -> [2, T, X] in base 62
```

So it will be stored like:

| id | shortern url | long url |
| -- | ------------ | -------- |
| 11157 | <www.test.com/2TX> | <www.longurl.com/awdasdwada/dsa>|

Base 62 ID length goes with ID. Also, Base 62 can be prediectable as 2TX + 1 represent a url when shortern url generated a lot. can have security issue, but the performance is good.

`Add cache layer`

add cache layer between server and DB can improve the performance. Choose like LRU cache expire strategy.
