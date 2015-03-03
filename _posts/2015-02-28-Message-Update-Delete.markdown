---
layout: post
title: Message Updates and Deletes
date:   2015-02-26
author: Jasdeep Jaitla
categories: pubnub update delete
---

PubNub stores all messages per channel with the Storage & Playback feature add-on. Since it stores all messages in a time series, and doesn't expose Updating and Deleting, you need a simple pattern to implement this functionality. It involves simply publishing the updated versions of messages within the stream. 

There are two ways to do this: 

  * [Interleaving](#interleaving)
  * [Side-Channel](#side-channel)

Let's use these two JSON messages as the messages we want to update and delete. The first message has a spelling error in the status, the second one was posted prematurely and needs to be deleted.

**message 1:**

{% highlight ruby %}
{
    message_id: 10001,
    channel: "jasdeep-status",
    user: "jasdeep",
    status: "Writng up design patterns",
    usecase: "update",
    deleted: false
}
{% endhighlight %}

**message 2:**

{% highlight ruby %}
{
    message_id: 20001,
    channel: "jasdeep-status",
    user: "jasdeep",
    status: "The Patriots lost the Super Bowl!",
    usecase: "delete",
    deleted: false
}
{% endhighlight %}


## Interleaving ##

Using the *Interleaving* pattern, you publish new versions of the same message to the same channel just like normal publishes. The subsequent messages must use the same message_id as the original message, and to simplify rendering, the messages for updates should contain the full content so that the original is not needed.

![Interleave](/images/interleave1.jpg)

Assuming we already published message 1 and 2 above. We would then publish the updated version (and deleted) to the same channel.

{% highlight javascript %}
pubnub.publish({
    channel: "jasdeep-status",
    message: {
        message_id: 10001,
        channel: "jasdeep-status",
        original_timetoken: 
        user: "jasdeep",
        status: "Writing up design patterns...",
        usecase: "update",
        deleted: false,
        is_update: true
    }
})
{% endhighlight %}

In this case we have updated part of the text as an additional publish to the channel, using the same message_id as the original message. In our client side logic, for displaying these messages we need to account for messages having the same ID and be able to update the content rather than display it as a new message.

#### Sample Message Tracking Code ####

While the code for updating display content varies considerably with each platform/UI  framework, a simple example for a single channel follows:

{% highlight javascript %}

var message_ids = {}
var deleted_ids = {};
var message_list = {};
var my_channel = "jasdeep-status";

function display_messages(message, channel) {
    (typeof message_list[channel] === 'undefined' ? message_list[channel] = {});
    (typeof message_ids[channel] === 'undefined' ? message_ids[channel] = []);
    (typeof deleted_ids[channel] === 'undefined' ? deleted_ids[channel] = []);
    
    // if new message, add to list
    if (message_ids[channel].indexOf(message.message_id) < 0) {
        message_ids[channel].push(message.message_id);
        message_list[channel][message_id] = message;
    }
    else {
        if (message.deleted) {
            deleted_ids[channel].push(message.message_id);

            // delete from message list
            delete message_list[channel][message_id];
            
            // update UI, remove message from display
            // ...
        }
        else {
            
            // replace content (same as adding new one, because of identical message_id)
            message_list[channel][message_id] = message;

            // update UI, update display content
            // ...
        }        
    }
}

pubnub.subscribe({
    channel: my_channel,
    message: function(msg, env) {
        display_messages(msg, my_channel);
    }
    error: function(e) {
        console.log(e);
    }
});

{% endhighlight %}

Of course if you are using a javascript frontend framework like Backbone or Angular, you can go directly to the Collection and search for the message_id to handle updates and deletes.

#### History Calls ####

With the Interleave pattern, it's not required to go further back in Storage History to see the original published message since it is simply replaced or deleted by subsequent publishes with the same message_id. The original message isn't needed to create the UI. 

If the original message is also retrieved through history, you process all the messages in order just like the subscribe handler above, updating content or removing content if the message_id is used again. Therefore interleaving update/delete messages makes it simpler for managing the UI code.

#### Final Note ####

If you are retrieving and displaying hundreds or thousands of messages from history it is possible that the rendering might be slow. In a scenario such as this where messages are being *displayed as they are being processed* it's possible to display a message that is later updated or deleted! Of course that will be corrected when the update/delete message is processed. 

To avoid this race condition, you can delay render until all messages have been processed, or you can use a Side Channel pattern described below.

## Side Channel ##

In the Side Channel pattern, you are publishing any updates and deletes to a separate channel that you also subscribe to. This means that you have a primary content channel and a side channel for updates/deletes.

In the Storage & Playback feature add-on, messages are stored on a per channel basis, this means that the side-channel has only update/delete messages in it, rather than both the orginal messages and the changes.

![Interleave](/images/sidechannel1.jpg)

When retrieving from Storage through history, before retrieving the main channel content you first retrieve all the updates/deletes from the side channel. Then you can process updates/deletes can be processed at the same time as processing the associated original message.

This pattern requires a little bit more code (albeit not much more) and alleviates the possibility in the Interleave pattern of displaying any original/older versions of messages before the updates/deletes are processed. 

This pattern might be a better chocie than Interleave if you are rendering as you go, have a slow rendering process, and potentially could have a race condition where the original message and the update/delete messages have a long time gap or a lot of other messages between them in the series causing delay before UI is updated to reflect updates/deletes.

Assuming we already published message 1 and 2 above. We would then publish the updated version (and deleted) to the same channel.

{% highlight javascript %}
pubnub.publish({
    channel: "jasdeep-status",
    message: {
        message_id: 10001,
        channel: "jasdeep-status-updates",
        original_timetoken: 
        user: "jasdeep",
        status: "Writing up design patterns...",
        usecase: "update",
        deleted: false,
        is_update: true
    }
})
{% endhighlight %}

#### Sample Message Tracking Code ####

While the code for updating display content varies considerably with each platform/UI  framework, a simple example for a single channel follows:

{% highlight javascript %}

var message_ids = {}
var deleted_ids = {};
var message_list = {};
var my_channel = "jasdeep-status";

function retrieve_update_deletes() {    
    pubnub.history({
        
    });
}

function display_messages(message, channel) {
    (typeof message_list[channel] === 'undefined' ? message_list[channel] = {});
    (typeof message_ids[channel] === 'undefined' ? message_ids[channel] = []);
    (typeof deleted_ids[channel] === 'undefined' ? deleted_ids[channel] = []);
    
    // if this is a new main channel message
    // and not in the deleted id's list, 
    // add to message list
    if (channel.indexOf("-UPDATES") === -1) {
    
       if (deleted_ids.indexOf(message.message_id) === -1) {
            message_ids[channel].push(message.message_id);
            message_list[channel][message_id] = message;    
       }       
    }
    // else it came from side-channel
    else {
        if (message.deleted) {
            deleted_ids[channel].push(message.message_id);

            // delete from message list
            delete message_list[channel][message_id];
            
            // update UI, remove message from display
            // ...
        }
        else {
            
            // replace content (same as adding new one, because of identical message_id)
            message_list[channel][message_id] = message;

            // update UI, update display content
            // ...
        }        
    }
}

// Subscribe to both the main content channel as well as the updates channel
pubnub.subscribe({
    channel: [my_channel, my_channel + "-UPDATES"],
    message: function(msg, env, channel) {
        display_messages(msg, channel);
    }
    error: function(e) {
        console.log(e);
    }
});

{% endhighlight %}

#### History Calls ####

With the Side Channel pattern, you want to retrieve history on the side channel first before retrieving the main history so that update/delete messages are processed at the same time as the original messages. 

#### Final Note ####

If you are retrieving and displaying hundreds or thousands of messages from history it is possible that the rendering might be slow. In a scenario such as this where messages are being *displayed as they are being processed* it's possible to display a message that is later updated or deleted! Of course that will be corrected when the update/delete message is processed. 

To avoid this race condition, you can delay render until all messages have been processed, or you can use a Side Channel pattern described below.
