---
layout: post
title: Inbound Channel Pattern
date:   2015-03-05
author: Jasdeep Jaitla
---

**Make sure you read this post all the way to the end!**

**The Hybrid Pattern is at the bottom of the post.**

PubNub stores all messages per channel with the Storage & Playback feature add-on. We have a history call for retrieving messages that may have been missed due to being disconnected. Using this is a common pattern, but what if you have hundreds of channels? It becomes far less efficient to call history on every single one. How do you change how you use channels so that you can reduce the number of history calls?

## 1-1 Chat ##

It's quite natural to think of making a new channel for each pair of users to have a 1-1 chat. If I have 4 different people I have private chats with, that means I have 4 channels I publish and subscribe to. This also means when I start the app after being disconnected for a few hours, I have 4 calls to history() to get any missed messages. After a few months, I have accumulated more friends on the app, and now I have 40 friends each with 1-1 chats. Now when I fire up my app, I have to make 40 history calls to see what messages were missed. This is starting to accumulate and become inefficient.

![Channels Per Chat]({{ site.baseurl }}/images/channels/channels1.jpg)

How can I change this so I don't need to make so many history API calls? Rethinking this - I want to reduce the history calls to be as few as possible. What I am trying to get by making the history calls is any messages that were *sent to me* in that time, because I always already have the messages that *I send*. What distinguishes one 1-1 chat from another is just who it's with. As far as the data itself, there is no difference in structure from one chat-channel to another.

With this in mind, let's restructure how chat messages are delivered.

![User Inbound Channel]({{ site.baseurl }}/images/channels/channels2.jpg)

In this scenario, each user has an Inbound channel for inbound messages from every chat/user. As you can see, both User B and User C are sending messages to User A's inbound channel. Likewise, when User A replies, they are sending the message to the corresponding users inbound channel in return.

This means that to get all messages missed for User A, that user only needs to call history on the inbound channel. This is very much like having an email address and an inbox. I have many conversations, but they all come into one place. My UI is what separates them logically, and it uses the metadata (the to/from/subject) to organize it in a way that I understand it, as separate conversations.

![Chat Inbound to User A]({{ site.baseurl }}/images/channels/channels3.jpg)

As you can see, Users B,C,D and E are all sending messages intended for separate 1-1 chats to User A's inbound channel. Just like 4 people sending me an email to my email address. Visually they will be separated in the UI, but as far as data is moved and organized, it's all coming in over a single channel, user a's inbound channel.

## Group Chat ##

In the case of group chat, when User A publishes a message to the group, that user can issue multiple publishes, one for each user in the group -- into their inbound channel. If the group is quite large, let's say more than 10 people in the group chat, I may want to separate that particular chat into a separate channel like the original scenario. In that case, I still have reduced my history calls significantly. 

## JSON Structure ##

In the original scenario, since each chat is a separate channel, I might have messages that look more like this:


**From User A to User B, sent to channel_1111**

{% highlight ruby %}
{
    message_id: 10001,
    author: "user_a",
    content: "What are you doing this weekend?",
    timestamp: 1425244035
}
{% endhighlight %}

In this Inbound Channel scenario, I might structure it a bit differently and add more metadata for the UI.


**From User A to User B, send to channel_user_b**

{% highlight ruby %}
{
    message_id: 10001,
    chat_id: "user_a_user_b",
    author: "user_a",
    content: "What are you doing this weekend?",
    timestamp: 1425244035
}
{% endhighlight %}

Of course, you can add a lot more information, information the UI would use to organize it visually, or to sort, or search, tag, etc. You have no limitations in what you can send. I've thrown in a "chat_id" key that helps my UI identify the chat it belongs to.

Lastly, I'll throw in a quick example of how it looks when do this Inbound method for a group chat.


**From User A to Group Chat, sent to channel_user_b and channel_user_c**

{% highlight ruby %}
{
    message_id: 10001,
    chat_id: "group_chat_1234",
    author: "user_a",
    content: "Did everyone buy tickets to the show yet?",
    timestamp: 1425244035
}
{% endhighlight %}

## Hybrid Pattern ##

One little gem that you can do in addition to the above pattern is to enable an interesting usage of PubNub. You can combine the two patterns. This allows for the ability to page back through history in a focused way and easier. First, you still use the Inbound channel, but you also use the Hybrid unique channel between each pair of users. When Users publish a message, they will dual-publish: Each User will publish to the Inbound channel and also publish to the Hybrid channel between them. This creates a record in Storage of the chat between the two Users in Hybrid channel. You don't need to Subscribe to the Hybrid channel, nor do you normally have to call history() on that Hybrid channel to get new messages, you only call history() on Hybrid when you want to view messages between those two Users and also you can of course go back in time by calling history() as you scroll for instance. 

![Inbound + Hybrid Channels]({{ site.baseurl }}/images/channels/channels4.jpg)

This allows you to more easily get the history of messages between two users, since the Inbound channel has messages mixed between all chats with every user that person is chatting with, it is harder to go back in time just to see only the messages between only that user and the user they were chatting with! So by publishing to Inbound and Hybrid channel you are simplifying the getting of new messages for all chats (Inbound), but also enabling going back in time to see messages between two users (Hybrid).

