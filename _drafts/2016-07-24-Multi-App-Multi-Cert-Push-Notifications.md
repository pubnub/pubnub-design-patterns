---
layout: post
title: Multi-App Multi-Cert Push-Notifications
date:   2016-07-24
author: Jasdeep Jaitla
---

The Mobile Push Gateway feature of PubNub creates a way to send Push Notifications embedded in Publishes. It's super convenient, particularly for Universal apps which is a single app designed for multiple device formats, and screen sizes.  However, when you have more than one App that needs to communicate together via PubNub *and* send Push Notifications to each, it gets complicated.  

Each **PubNub Keyset** \[publish + subscribe + secret key\] can only have *one* of each of these: 
 
* API Key for Google Cloud Messaging (GCM) 
* Apple Push Certificate

\[ FYI: GCM is now called Firebase Cloud Messaging, FCM, and they make it more confusing by making it seem like it's PubNub-like, but it's not, it's still the same Push Notification system. \]

When you have *two* apps, you have *two* of each of those Push Notification certs, so how can you communicate between the apps and also send Push Notifications to both apps? The answer is by using *two* PubNub Keysets and following a design pattern. 

Lets illustrate two such patterns, each solve the problem, but are slightly different in approach. For convenience, we'll use the Taxi App paradigm: there are two apps on each platform Store \[iOS,Android\], one being the Driver App, the other being the Passenger App. 

The two patterns are called:

* <a href="#single">Single Keyset for Pub/Sub</a>
* <a href="#bouncing">Keyset Bouncing for Pub/Sub</a>

### <a id="single" name="single"></a>Single Keyset for Pub/Sub ##
 
First, you need to set up two Keysets, so in the PubNub Admin, under your App, create two Keysets (publish/subscribe/secret keys). **Note**: If you are also doing several environments, like development, staging/testing, and production, you will need two Keysets for each environment. 

**Keyset 1** - Used for all publish/subscribe for both Passenger and Driver Apps

**Keyset 1** - Passenger app device token/id registration on trip channel when it’s created for chat between passenger/driver; Apple Push Cert, GCM API Key setup for Passenger app.

**Keyset 2** - Never used for subscribe, but you will publish to this when you want to send Push Notifications to Drivers

**Keyset 2** - Driver app device token/id registration on channels; Apple Push Cert, GCM API Key setup for Driver app.







###  <a id="bouncing" name="bouncing"></a>Keyset Bouncing for Pub/Sub ##

