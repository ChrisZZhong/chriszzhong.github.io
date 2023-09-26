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

It also integrates well with frameworks like `Spring Web MVC` (or `Spring Boot`), as well as with standards like `OAuth2`.

## main concepts

- `Authentication` is the process of verifying `who you are`. For example, when you log in to a website, generally you are asked to enter your username and your password to verify your identity.

- `Authorization` is the process of verifying the `permissions` user have. With Authentication, the server only knows who you are. With Authorization, the server knows what permissions you have.

- `Servlet Filters` are a `bunch of classes` that can be `chained together` and that `intercept` incoming requests and outgoing responses.

### Spring Servlet Filters

The authentication and authorization should be done **before** a request hits your @Controllers, we need to put filters in front of servlets. You could think about writing a `SecurityFilter` that intercepts all requests and checks whether the user is authenticated and authorized to access the requested resource.

A SecurityFilter could look like this:

```java
import javax.servlet.*;
import javax.servlet.http.HttpFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

public class SecurityServletFilter extends HttpFilter {

    @Override
    protected void doFilter(HttpServletRequest request, HttpServletResponse response, FilterChain chain) throws IOException, ServletException {

        UsernamePasswordToken token = extractUsernameAndPasswordFrom(request);  // (1)

        if (notAuthenticated(token)) {  // (2)
            // either no or wrong username/password
            // unfortunately the HTTP status code is called "unauthorized", instead of "unauthenticated"
            response.setStatus(HttpServletResponse.SC_UNAUTHORIZED); // HTTP 401.
            return;
        }

        if (notAuthorized(token, request)) { // (3)
            // you are logged in, but don't have the proper rights
            response.setStatus(HttpServletResponse.SC_FORBIDDEN); // HTTP 403
            return;
        }

        // allow the HttpRequest to go to Spring's DispatcherServlet
        // and @RestControllers/@Controllers.
        chain.doFilter(request, response); // (4)
    }

    private UsernamePasswordToken extractUsernameAndPasswordFrom(HttpServletRequest request) {
        // Either try and read in a Basic Auth HTTP Header, which comes in the form of user:password
        // Or try and find form login request parameters or POST bodies, i.e. "username=me" & "password="myPass"
        return checkVariousLoginOptions(request);
    }


    private boolean notAuthenticated(UsernamePasswordToken token) {
        // compare the token with what you have in your database...or in-memory...or in LDAP...
        return false;
    }

    private boolean notAuthorized(UsernamePasswordToken token, HttpServletRequest request) {
       // check if currently authenticated user has the permission/role to access this request's /URI
       // e.g. /admin needs a ROLE_ADMIN , /callcenter needs ROLE_CALLCENTER, etc.
       return false;
    }
}
```

1. First, the filter needs to extract a username/password from the request. It could be via a Basic Auth HTTP Header, or form fields, or a cookie, etc.

2. Then the filter needs to validate that username/password combination against something, like a database.

3. Then the filter needs to check whether the user has the proper permissions to access the requested resource.

4. If all checks pass, the filter can let the request go through to the Spring DispatcherServlet and the @Controllers/@RestControllers.

## FilterChain

however, you would split this one filter up into multiple filters, that you then chain together. EG: `AuthenticationFilter`, `AuthorizationFilter`, `LoggingFilter`, `CachingFilter`, etc. break it into several filters has sepecific responsibility.

## Authentication

### OAuth2

OAuth2 is a standard that allows users to grant access to their data to third parties. For example, you can grant access to your Google Calendar to a third party, so that they can read your calendar and display it in their app.

### JWT (JSON Web Token)

JWT is a standard that allows you to encode data in a JSON format. It is often used to encode authentication information, like the username, the permissions, etc. It is often used in combination with OAuth2.

JWT mainly contains three parts:

- Header: contains information about the type of token and the hashing algorithm used to sign the token.

- Payload: contains the claims, which are statements about an entity (typically, the user) and additional data. Claims can be things like the username, the permissions, etc.

- Signature: is used to verify that the sender of the JWT is who it says it is and to ensure that the message wasn’t changed along the way. (Created by hashing the header and the payload with a secret key)
