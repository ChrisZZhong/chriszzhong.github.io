---
layout: post
title: "Srping Security and Authentication"
date: 2023-04-11
description: "Srping Security and Authentication"
tag: Spring Framework
---

# Spring Security

## Spring Security introduction [More](https://www.marcobehler.com/guides/spring-security)

`Spring Security` is a `bunch of servlet filters` that help you add `authentication` and `authorization` to your web application.

## talk about details how authentication and authorization works, login/ not login, permission/role, etc. (Interview question)

## Spring Security Flow

### OAuth2

OAuth2 is a standard that allows users to grant access to their data to third parties. For example, you can grant access to your Google Calendar to a third party, so that they can read your calendar and display it in their app.

### JWT (JSON Web Token)

JWT is a standard that allows you to encode data in a JSON format. It is often used to encode authentication information, like the username, the permissions, etc. It is often used in combination with OAuth2.

JWT mainly contains three parts:

- Header: contains information about the type of token and the hashing algorithm used to sign the token.

- Payload: contains the claims, which are statements about an entity (typically, the user) and additional data. Claims can be things like the username, the permissions, etc.

- Signature: is used to verify that the sender of the JWT is who it says it is and to ensure that the message wasn’t changed along the way. (Created by hashing the header and the payload with a secret key)
