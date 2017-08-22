---
layout: post
title: "Nunit Assert"
date: 2016-08-22 20:55:39 +1000
comments: true
categories: nunit
keywords: Nunit
description: 
---
## Assert.AreEqual vs Assert.AreSame
Very frequently I use Assert.AreEqual and Assert.AreSame for doing assertions in the code. Below is high level difference between both.

### Assert.AreSame
Assert.AreSame checks whether both comparing objects are exactly the same ( reference indicate same object in memory) .It is normally known as Reference Equality

### Assert.AreEqual
Assert.AreEqual checks whether both objects contain same value. It is normally known as Value Equality. For primitive value types ( like int, bool) this is straight forward. But for other types ( especially user defined objects) , it is depends on how the type defines equality. 

Hence Assert.AreEqual will fail ( most of the time) when we compare two objects. This [link](https://github.com/nunit/nunit/issues/730) have some discussion point about the same. 