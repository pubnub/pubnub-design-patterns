---
layout: post
title: Flexible History Calls
date:   2015-03-08
author: Jasdeep Jaitla
---

Our Javascript History API allows you options for getting the time series data in a channel in flexible ways but always gives messages in 100 count pages.
I wrote a simple wrapper that extends this functionality to handle the actual paging to retrieve messages based on time, counts, etc.
One difference is that it retrieves and returns results in an "all-at-once" fashion. That means that if you retrieve many pages of history (i.e 100+
messages) then it doesn't execute callback until all messages are retrieved.

The wrapper accepts a few parameters:

* **last**: retrieve the latest n messages
* **since**: retrieve messages since a timetoken
* **between**: retrieve messages between timetokens
* **at**: retrieve a message at a timetoken
* **getrange**: get the date range of first and most recent messages in a channel

In all the methods, it accepts both Unix epoch timestamps and PubNub 17-digit timetokens as strings or integers and converts automaticall.

The Github repository with all the code, and instructions in the README are here:
[https://github.com/scalabl3/pubnub-flex-history](https://github.com/scalabl3/pubnub-flex-history)

## Usage Summary ##

Pretty straightforward here:

* Get the PubNub Javascript SDK
* Get the pubnub_flex_history from this repo (CDN url provided via rawgit)
* Add the method to your instantiated pubnub object

{% highlight html %}

<script src="//cdn.pubnub.com/pubnub-3.7.8.js"></script>
<script src="//cdn.rawgit.com/scalabl3/pubnub-flex-history/v1.01/pubnub-flex-history-min.js"></script>

<script>
  // Call Init first to create a PubNub instance, then add the wrapper method to that object
  var p = PUBNUB.init({
    publish_key: 'demo',
    subscribe_key: 'demo'
  });

  // ** REQUIRED ** Add flex_history method to your PubNub object
  p.flex_history = pubnub_flex_history;

  // Example of a generic callback, but of course you can use your own
  var flex_history_callback = function(result) {
    if (!result.error) {
      console.log(result.operation + " completed", result);
    }
    else {
      console.warn(result.operation + " failed", result);
    }
  }
</script>

{% endhighlight %}

## Usage Options ##

### general ###

The general usage follows as:

    p.flex_history(options_object, completed_callback)

options_object requires a channel name, and a command which is one of [last, since, between, at, getrange]

{% highlight ruby %}
{
  channel: [channelname]
}
{% endhighlight %}

### last ###

Gets the last n messages from the channel.

{% highlight javascript %}

// get last 30 messages
var options = {
  channel: 'AAPL',
  last: 30
}

p.flex_history(options, flex_history_callback);

{% endhighlight %}

### since ###

Get all messages since epoch timestamps or PubNub timetoken.

{% highlight javascript %}

var since = 1426010693;

var options = {
  channel: 'AAPL',
  since: since
}

p.flex_history(options, flex_history_callback);

{% endhighlight %}

### between ###

Get all messages between epoch timestamps or PubNub timetokens.

{% highlight javascript %}

var options = {
  channel: 'AAPL',
  between: [1426010693, 1426021664]
}

p.flex_history(options, flex_history_callback);

{% endhighlight %}

### at ###

Get single nearest or exact message at epoch timestamp or PubNub timetoken.

{% highlight javascript %}

var options = {
  channel: 'AAPL',
  at: 14259785889989920
}

p.flex_history(options, flex_history_callback);

{% endhighlight %}

### getrange ###

Get the start and end DateTime range of the channel, timetoken of first message and timetoken of most recent message.

{% highlight javascript %}

var options = {
  channel: 'AAPL',
  getrange: true
}

p.flex_history(options, flex_history_callback);

{% endhighlight %}