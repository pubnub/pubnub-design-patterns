---
layout: post
title: Server Ingress & Fan-In
date:   2016-01-11
author: Jasdeep Jaitla
---

Data Aggregation and collection is a common need for PubNub customers. 
There are a few different ways to approach this, but they all share the same core concepts. 
The primary concern is ensuring you can receive incoming messages at high volume without missing messages due to overflow.


