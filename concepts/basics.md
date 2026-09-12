# React Basic Concepts

## Basic Keywords

### DOM(Document Object Model)

a programming interface that represents a web page’s HTML structure as a tree of objects in memory,
allowing JavaScript to access, modify, and dynamically manipulate the page's content, style, and structure.

### Virtual DOM(Document Object Model)

a lightweight, in-memory representation of the Real DOM. Instead of interacting directly with the browser's
actual HTML structure—which is a slow and resource-heavy operation—React creates a copy of the UI layout using
plain JavaScript objects.

### Imperative Programming

a style of programming in which you describe how a program should accomplish a task step by step.

### Declarative Programming

a style of programming in which you describe what you want the program to accomplish without describing how.
This system is always built on top of an imperative one. Essentially this means running a program/function
that has already been established.

### Recursion

when a function calls itself. Each call to itself takes up a space in memory.

### Traverse

to move across or through something. In the case of a tree data structure we mean to move from element
to element, that is from parent to child to sibling, etc.

### POJO (Plain Old Javascript Objects)

simple collections of key/value pairs

### Component

In React, a function component is a function that returns a React Element (which may contain other React Elements).
It is intended to be called by React. Components, by their nature, are reusable, but you likely won't
reuse every one.

### JSX(JavaScript XML)

a syntax extension for JavaScript used primarily with React to describe what the user interface (UI) should look like.
While it looks remarkably like HTML, it possesses the full power of JavaScript and compiles down to regular
JavaScript objects under the hood by a transpiler like babel
