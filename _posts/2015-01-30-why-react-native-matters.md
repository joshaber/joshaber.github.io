---
layout: post
title: Why React Native Matters
---

A couple days ago Facebook announced React Native, a version of [React](http://facebook.github.io/react/) that outputs native views instead of a DOM.

This is fantastic news for us native developers. But before we get to why, let's dispell a couple concerns.

### React Native isn't just another Titanium or PhoneGap.

React Native is **not** a write-once-run-everywhere solution. The phrase Facebook has used so far is "learn-once-write-everywhere." They're interested in sharing the paradigm of React across platforms, not the code necessarily.

### JavaScript is an implementation detail.

Don't get distracted by the use of JavaScript. JavaScript is just a language. There are plenty of [other](http://www.purescript.org), [better](http://www.typescriptlang.org) languages that can be compiled to JavaScript.

The important thing about React Native is the *idea* behind it.

### React lets us write our UIs as a pure function of their state.

This is a big, important idea.

Right now we write UIs by poking at them, manually mutating their properties when something changes, adding and removing views, etc. This is fragile and error-prone. [Some tools](https://github.com/ReactiveCocoa/ReactiveCocoa) exist to lessen the pain, but they can only go so far. UIs are big, messy, mutable, stateful bags of sadness.

React let us describe our entire UI for a given state, and then it does the hard work of figuring out what needs to change. It abstracts all the fragile, error-prone code out away from us. We describe what we want, React figures out how to accomplish it.

UIs become composable, immutable, stateless value types. React Native is fantastic news.
