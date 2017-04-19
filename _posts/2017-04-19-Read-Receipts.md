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

## Multi-device Read State ##


![Basic]({{ site.baseurl }}/images/readreceipts/rr-3.jpg)


If you have any questions, please don't hesitate to make contact!
