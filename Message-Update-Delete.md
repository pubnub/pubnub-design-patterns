title: Message Updates and Deletes

# Pattern: Message Updates and Deletes  #

PubNub stores all messages per channel with the Storage & Playback feature add-on. Since it stores all messages in a time series, and doesn't expose Updating and Deleting, you need a simple pattern to implement this functionality. It involves simply publishing the updated versions of messages within the stream. 

There are two ways to do this: interleaving and sidechannel approaches. 

Let's use these two JSON messages as the messages we want to update and delete. The first message has a spelling error in the status.

**message 1:**

```javascript
{
    message_id: 10001,
    channel: "jasdeep-status",
    user: "jasdeep",
    status: "Writng up design patterns",
    usecase: "update",
    deleted: false
}
```

**message 2:**

```javascript
{
    message_id: 20001,
    channel: "jasdeep-status",
    user: "jasdeep",
    status: "The Patriots lost the Super Bowl!",
    usecase: "delete",
    deleted: false
}
```

### Interleaving ###

Using the *Interleaving* pattern, you publish new versions of the same message to the same channel just like normal publishes, but the subsequent messages also use the same message_id as the original message. 


Assuming we already published message 1 and 2 above. We would then publish the updated version (and deleted) to the same channel.

```javascript
pubnub.publish({
    channel: "jasdeep-status",
    message: {
        message_id: 10001,
        channel: "jasdeep-status",
        user: "jasdeep",
        status: "Writing up design patterns...",
        usecase: "update",
        deleted: false
    }
})
```

In this case we have updated part of the text as an additional publish to the channel, using the same message_id as the original message. In our client side logic, for displaying these messages we need to account for messages having the same ID and be able to update the content rather than display it as a new message.

While the code for updating display content varies considerably with each platform/UI coding framework, a simple example for a single channel follows:

```javascript

var displayed_message_ids = {}

function display_messages(message, channel) {
    displayed_message_ids[channel] = [];
}

pubnub.subscribe({
    message:     
    
});

```

### Side Channel ###

