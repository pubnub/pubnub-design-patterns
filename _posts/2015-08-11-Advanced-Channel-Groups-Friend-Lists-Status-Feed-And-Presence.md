---
layout: post
title: Friend Lists and Presence
ltitle: Advanced Channel Groups 
stitle: Friend Lists, Status Feed and Presence
date:   2015-08-11
author: Jasdeep Jaitla
---

Using the [Stream Controller](https://www.pubnub.com/products/stream-controller/) add-on and the 
[Presence](https://www.pubnub.com/products/presence/) add-on, you can create simple Friend Graphs as well 
as a Status message feed. In the case of the Friend Graph, it's a simple Follow/Followers model. Anything more 
complex, such as a true multi-degree graph or an application that requires querying/traversing graphs, 
might be better served with a specialized implementation that uses a graph database such as Neo4J or other
more sophisticated data mechanisms, but to be honest, these types of use cases are fairly uncommon. 

The essential feature required to implement this is the "Channel Group" feature, which can also be 
thought of as a "Subscribe Group." The first concept to understand is what a Channel Group does. It groups 
channels into a persistent collection of channels that you can modify dynamically, and it allows you to subscribe
to all these channels by simply subscribing to the group.. 

On the PubNub server side, all published messages and presence events are aggregated for all channels in the channel 
group. What this means  is that if you subscribe to the Channel Group, you get all messages from all the channels in 
the group in a single stream, if you subscribe to the presence events of the Channel Group, you get all the presence 
events for all the channels in the channel group in a single stream.

It's this multiplexing magic that allows for creating Friend Lists and Status Feeds. 

## User Identification

All Presence features use the UUID that is set on PubNub client initialization for tracking of that client,
which typically is the user. While it's called a UUID, it doesn't have to follow the standard UUID format at all,
it's a freeform string. The only real requirement (also not enforced), is that it's uniqe. 

It's recommended to actually combine a unique identifier with the platform as well for instance:

* 0c2340c2-8cc1-4898-8868-444ba77d02d2::ipad
* 0c2340c2-8cc1-4898-8868-444ba77d02d2::web
* 0c2340c2-8cc1-4898-8868-444ba77d02d2::android

This way, not only do you track the user because you can see the uuid, but if the user is also using multiple
platforms, you can see that too!

If you don't supply your own UUID to PubNub init, one will be generated, but it may not be reused or saved 
(depending on the platform), so you will want to set your own. Again it can follow any format you want.

{% highlight javascript linenos %}

// Initialize with UUID (from successful authentication response with backend)
var p = PUBNUB.init({
  publish_key: "...",
  subscribe_key: "...",
  uuid: "0c2340c2-8cc1-4898-8868-444ba77d02d2::web"
});

{% endhighlight %}

In the case of the examples below, for readability, I am just going to use User A, User B, etc. and follow 
a convention of ```ch-user-a-[xyz]``` for channels, and ```cg-user-a-[xyz]``` for channel groups.

{% highlight javascript linenos %}

// Initialize with UUID (from successful authentication response with backend)
var p = PUBNUB.init({
  publish_key: "...",
  subscribe_key: "...",
  uuid: "user-a"
});

{% endhighlight %}

## User Channels

Every user will need two channels: 1) a channel they publish to for their own status messages, 2) a 
channel they subscribe to to indicate they are online. They will be unique for each user. For convenience,
we will use user-[letter], like user-a or user-b, as the unique identifier.
  
We will follow this naming convention:

* ch-user-a-status (publish)
* ch-user-a-present (subscribe)


![User Channels]({{ site.baseurl }}/images/friendlist/friendlist1.jpg)

These channels will be added into Channel Groups to enable the monitoring and aggregation of both
status messages into a Feed, as well as presence for online status. It's not required to do both if you 
only want to support Friend Lists/Presence or you only want to support Status Feeds.

{% highlight javascript linenos %}

// Publish a status message
p.publish({
  channel: "ch-user-a-status",
  message: { author: "user-a", status: "I am reading about Advanced Channel Groups!", timestamp: Date.now() / 1000 },
  callback: function(m) {
    console.log("PUB: ", m);
  }
});

{% endhighlight %}


## User Channel Groups

Each user will also have two channel groups: 1) a channel group for observing the online status of friends, 2)
a channel group to receive status updates in realtime. Again we'll use the same convention for unique user
identifiers.

We will follow this naming convention:

* cg-user-a-friends
* cg-user-a-status-feed

![Channel Groups]({{ site.baseurl }}/images/friendlist/friendlist2.jpg)

Creating channel groups requires you to add at least one channel to the group. This means that when you set up
a user. The easiest way to do this is to simply add the "present" channel to each group. The example is in 
Javascript, but typically you call these API methods from your backend server when a user registers for an account.

{% highlight javascript linenos %}

// Add ch-user-a-present to cg-user-a-friends
p.channel_group_add_channel({
  channel_group: "cg-user-a-friends",
  channel: "ch-user-a-present",
  callback: function(m) {
    console.log("CG-Add: ", m);
  }
});

// Add ch-user-a-present to cg-user-a-status-feed
p.channel_group_add_channel({
  channel_group: "cg-user-a-status-feed",
  channel: "ch-user-a-present",
  callback: function(m) {
    console.log("CG-Add: ", m);
  }
});

{% endhighlight %}

## Friending

Expanding the Friend graph through friending is very straightforward: You are simply adding channels
to channel groups! In the case of User A and User B becoming friends, you are adding the "present"
channel to each users' friends group, and adding the "status" channel to each users' status-feed group.
Again, this is in Javascript, but you will want to call these API methods from your backend server
when you receive a REST request that two users are friending each other.

![Friending]({{ site.baseurl }}/images/friendlist/friendlist3.jpg)

{% highlight javascript linenos %}

// ************************************
// * User A and User B become friends
// ************************************

// Add User B to User A's groups: Add ch-user-b-present to cg-user-a-friends
p.channel_group_add_channel({
  channel_group: "cg-user-a-friends",
  channel: "ch-user-b-present",
  callback: function(m) {
    console.log("CG-Add: ", m);
  }
});

// Add User B to User A's groups: ch-user-b-status to cg-user-a-status-feed
p.channel_group_add_channel({
  channel_group: "cg-user-a-status-feed",
  channel: "ch-user-b-status",
  callback: function(m) {
    console.log("CG-Add: ", m);
  }
});

// Add User A to User B's groups: Add ch-user-a-present to cg-user-b-friends
p.channel_group_add_channel({
  channel_group: "cg-user-b-friends",
  channel: "ch-user-a-present",
  callback: function(m) {
    console.log("CG-Add: ", m);
  }
});

// Add User B to User A's groups: ch-user-a-status to cg-user-b-status-feed
p.channel_group_add_channel({
  channel_group: "cg-user-b-status-feed",
  channel: "ch-user-a-status",
  callback: function(m) {
    console.log("CG-Add: ", m);
  }
});

{% endhighlight %}

Now to see these working, it comes to how you subscribe. What's a bit different here
is that you will subscribe to one group for messages (status-feed) but subscribe to 
the other groups's Presence Event channel for the online/offline status (friends).

## Friends Online/Offline (Presence)

For Presence, we track the subscribers on a channel, and we create a *sister* channel
based on the channel name for all the Presence Events on the main channel. This *sister*
channel is simple the channel name + "-pnpres". So for channel ```ch-user-a-present```, the
sister channel is ```ch-user-a-present-pnpres``` and that's where PubNub publishes Presence
Events that occur on ```ch-user-a-present```. The Presence Events are as follows:
 
* JOIN - client subscribed
* LEAVE - client unsubscribed
* TIMEOUT - client disconnected without unsubscribing after timeout period
* STATE-CHANGE - client changed the contents of the state object

and for each event, we also include the UUID and the channel occupancy (how many subscribers).

To see your friends online/offline status and be updated in realtime, you subscribe
to the friends channel group, but not directly, you subscribe to the Presence Event
*sister* channel only, by appending "-pnpres" to the channel group name. 

The reason you don't want to subscribe directly to the group is because Channel Groups are
"Subscribe" groups, and therefore you would inadvertently subscribe to all that users' 
friends' "present" channels! We are only interested in the Presence Events in this group.

![Presence]({{ site.baseurl }}/images/friendlist/friendlist4.jpg)

{% highlight javascript linenos %}

// Get the List of Friends
p.channel_group_list_channels({
  channel_group: "cg-user-a-friends",
  callback: function(m) {
    console.log("FRIENDLIST: ", m);
  }
});

// Which Friends are online right now
p.here_now({
  channel_group: "cg-user-a-friends",
  callback: function(m) {
     console.log("ONLINE NOW: ", m);
  }
});

// Watch Friends come online / go offline
p.subscribe({
  channel_group: "cg-user-a-friends-pnpres",
  message: function(m,a,b,c) {
    console.log("FRIEND PRESENCE: ", m,a,b,c);
  }
});

{% endhighlight %}


## Status Feed (Messages)

The status-feed channel group is much more straightforward, you are going to subscribe directly to the
channel group and you will receive status updates in realtime via each channel in the group.

![Status Feed]({{ site.baseurl }}/images/friendlist/friendlist5.jpg)

Since we include the "present" channel for this user in the channel group (```ch-user-a-present```), this also has the 
net effect of subscribing to that channel, it also means it generates a JOIN event for every channel group that 
includes this user's present channel. So, if User B is friends with User A, when User A subscribes to this status-feed
channel group, it also subscribes to User A's present channel and generates that JOIN event, as well as all the
other Presence events. 

**NOTE**: If you implementing only the Friend List and not the Status Feed, User A will need to subscribe to this 
```ch-user-a-present``` channel directly since User A isn't subscribing to the status feed group which includes
this channel.

{% highlight javascript linenos %}

// Get Status Feed Messages
p.subscribe({
  channel_group: "cg-user-a-status-feed",
  message: function(m,a,b,c) {
    console.log("STATUS: ", m,a,b,c);
  }
});

{% endhighlight %}

## Summary

Simple and powerful Friend Graphs are fairly easy to do with PubNub! I'd argue it's even easier than trying
to develop a full backend to support the Presence and Status Feeds, and on top of that, it's realtime. Can't be
beat! 

![Summary]({{ site.baseurl }}/images/friendlist/friendlist6.jpg)

Lastly is retrieving history of status messages can be a bit more work as you have to retrieve messages from each 
channel in the group individually (each friend) and mash/sort them together client side. I'll make another post
on how to do that, and a little bit of code that can handle it at least semi-auto-magically. 

## Final Notes

For every decision there is a counter-decision, and for every idea there is a more complex version. 
I'll throw in a few caveats to this implementation, there are some interesting ways of changing Status Feeds that can 
make things a bit trickier, one is "weighting" the status message, so that it's no longer chronological, but rather
based on interests, or activity, or other things. Another layer of complexity that you can add is commenting on status
items. Again, it requires a bit more logic here, and it might not be the simplest thing to do, but it depends on
how you want it to work. It's certainly quite possible, but not quite *as easy* as the above.

Happy coding!