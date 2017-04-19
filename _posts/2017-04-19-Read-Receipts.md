---
layout: post
title: Read Receipts
ltitle: Read Receipts
stitle: Single and Multichannel Message State
date:   2017-04-19
author: Jasdeep Jaitla
---

Keeping track of message state in chat or other applications follows a very straightforward pattern:

> The recipient must notify the sender that they received the message.

In the case of PubNub (and most chat and email systems, even UPS and FedEx for that matter)
 this means the recipient must publish a message back to the sender on the channel the
 sender is listening to. In the most basic case that means for every message received you
 publish back a "receipt."

![Basic]({{ site.baseurl }}/images/readreceipts/rr-1.jpg)

In this diagram, when the recipient receives a message with the msgID, it publishes back a "receipt" message to the sender
including that received msgID.

## Timestamp Based ##

Publishing a message back for every message received isn't necessarily the most efficient. While it does acknowledge
 every single message individually, it doubles overall publishing volume. It's not necessarily as important either that
 the receipt goes back immediately (in under 250ms of receiving a message).

Instead of sending back a receipt for every message then, one can send back a timestamp and optionally an array of msgID's.
Every message that is older than the timestamp provided will be marked as read (or each msgID can be processed if that is preferred).
This allows for a smaller ratio of published receipts to published messages.

![Basic]({{ site.baseurl }}/images/readreceipts/rr-2.jpg)

In this diagram, the recipient is not sending a receipt for every message received, instead just a timestamp and optionally an
 array of msgID's that have been received since the last receipt. In addition, these can be buffered by time, meaning you don't
 send the receipt immediately, but rather, wait n seconds to see if any other messages are received before sending the receipt.
 This can reduce the receipt-publish:message-publish ratio and save a lot on message volume.

## Multi-Device Read State ##

When you have multiple interfaces that can be using same application simultaneously, such as iPhone, iPad, Android, Browser, etc.
 then keeping the notifications & messages "read state" of various chats and channels can make an application seem much more intelligent.
When you look at a chat on one interface, essentially marking it as read, if you then still see it as unread on a second interface
 it seems "broken."

In order to keep this in sync, it's simply a matter of keeping track via a channel for each user.
We can use something like: `user.[userID].readState` Each user subscribes and publishes a JSON dictionary of **timestamps** for each \[channel,
conversationID, userID\] that they participate in.

Every message that is older than these timestamps will be "unread", but in case you are using Inbound Channel design pattern,
it's not necessarily based on channels but *conversations*. This allows you to have individual conversations have a read state, rather than
just chronology alone.

![Basic]({{ site.baseurl }}/images/readreceipts/rr-3.jpg)

In this diagram, the JSON dictionary of conversations or channels have timestamps, and any message within these channels or conversations
that is older than the timestamp is unread. Every time a change occurs (i.e. a user taps or clicks on a conversation) this dictionary is updated
 and published. When receiving the dictionary, the logic should be to only replace existing timestamps with newer timestamps in order to prevent race conditions.
 To avoid an endless loop, don't publish an updated dictionary as a result of receiving one or you'll end up bouncing them back and forth!

If you have any questions, please don't hesitate to make contact!
