---
layout: post
title: "JavaScript questions"
date: 2023-08-07
description: "JavaScript"
tag: Interviews
---

## What is ES6?

ECMAScript 6 is also known as ES6. It is used to create web applications. It is a programming language based on scripts that supports `object-oriented` and `functional` programming styles.

ECMA is the standard, JS is the language in practice

## What is Hoisting?

Hoisting is a `mechanism`.

Before execution, all function and variable declarations are moved to top and scanned. So you can use a function or variable before declaring it without error.

“Function declaration” only applies to the keyword function declaration since the function expression & arrow function are assigned to variables.

## What are the differences between Var, Let, and Const?

- let are only available inside the block where they're defined.

- var are available throughout the function in which they're declared, if defined outside a function block, it is global value.

- Const are used to define values that are not changed.

## What is the difference between == and ===?

- == only compare values.

- === will check datatype and compare two values.

## What does `this` refer to in JavaScript?

For object : refer to the instance of the object.

For Keyword Function : ‘this’ changes based on where it is invoked

For Arrow Function : in the scope where this is defined
