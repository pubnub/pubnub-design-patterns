---
layout: post
title: Friend Lists and Presence
date:   2015-08-11
author: Jasdeep Jaitla
---

Creating a simple friend graph and being able to monitor presence and create a simple (unranked)
status message feed is straightforward with PubNub using the Stream Controller add-on. In the Stream
Controller add on is the "Channel Group" feature, which can also be thought of as a "Subscribe Group."

The first concept to understand is what a Channel Group does. It groups channels into a persistent 
collection of channels that you can modify dynamically. On the PubNub server side, all messages
published to any of the channels in the Channel Group get multiplexed together into a single stream
to be received by subscribers to that Group.

In regard to Presence, Channel Group subscriptions will aggregate all the Presence events for each
channel in the group to the Channel Group subscriber, basically multiplexing the event notifications
for each channel to a Channel Group level notification delivery.

This is the magic that allows for creating Friend lists and status feeds. First you need to have a 
Channel Group for each User. You also need a channel that they always subscribe to to indicate they
are online. We add this to the Channel Group so when they subscribe to the group they also subscribe
by default to this channel. This "presence" channel, or "user is present" channel is what you add to 
that users friends' channel groups so the friends can see them when they are online.

![Channels]({{ site.baseurl }}/images/friendlist/friendlist2.jpg)

For Status updates, or outbound messages authored by the user, you have a "publish" or outbound channel 
for each user as well. Any time they want to make a new post, you publish it to this channel. If this channel 
is added to that users friends' channel groups, they will each receive this message.

![ChannelGroups]({{ site.baseurl }}/images/friendlist/friendlist1.jpg)

"Friending" is simply adding the "present" channel to the friend's channel group for Presence, or the 
"publish" channel to the friend's channel group for status updates, or both if you want both. When
the friend subscribes to their channel group they will see presence and status updates since their
channel group will have each of their friends' publish and present channels in the group.

![ChannelGroups]({{ site.baseurl }}/images/friendlist/friendlist3.jpg)

![ChannelGroups]({{ site.baseurl }}/images/friendlist/friendlist4.jpg)

As you can see, this makes the process of having a Friend List and sharing status and Presence in
simple Friend graph's very easy to do. What is a bit more complex is to rank/prioritize
status updates in the UI in a non-chronological way. This is still possible with client side code, but 
not quite as straightforward. 

To get the friend list of any user, you simply call the list_channels method for a channel group,
in Javascript it's called channel_group_list_channels({...}), you specify the channel group name,
and then looking at the channels listed, you can easily see who the friends are.