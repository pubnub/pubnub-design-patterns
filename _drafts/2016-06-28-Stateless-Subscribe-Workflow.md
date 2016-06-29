---
layout: post
title: Stateless Subscribe Workflow
date:   2016-01-10
author: Jasdeep Jaitla
---

PubNub Subscribe is a stateless system. We do not track cursors for subscriber connections or keep any state. So how do we deliver messages that are new? The key to how it works is the use of PubNub timetokens. 

Let's look at how published messages are handled on PubNub Servers. When you publish a message it gets replicated globally and goes into a Message-Queue (MQ) with a fixed size (100 messages). These don't stay there forever, each message lasts for roughly 10-15 minutes, typically the latter. The time can be on the shorter side during peak usage times.

![Message-Queue]({{ site.baseurl }}/images/subscribeflow/1-message-buffer.jpg)

When you do a first `subscribe`, it's called a Timetoken/0, basically subscribing with a timeoken=0. PubNub servers return a timetoken, and the subscribe connection is re-established with a `subscribe_with_timetoken` call using the returned timetoken. It is now ready to receive messages.

You can also subscribe with a timetoken in your first subscribe, this will return any messages from the MQ.

{% highlight javascript %}
// Subscribe with Timetoken from 60 seconds ago
p.subscribe({
    channel: "mychannel",
    timetoken: (Date.now() - 60) * 10000000,
    message: function(msg, env, a, b) {
    console.log("Message Received", msg);	
  }
}	
{% endhighlight %}

![Message-Queue]({{ site.baseurl }}/images/subscribeflow/2-subscribe-timetoken.jpg)