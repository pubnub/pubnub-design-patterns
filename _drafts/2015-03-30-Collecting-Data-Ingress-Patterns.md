---
layout: post
title: Collecting Data -- Ingress Patterns
date:   2015-03-30
author: Jasdeep Jaitla
---

Data Aggregation and collection is a common need for PubNub customers. There are a lot of different ways to approach this, and several high level patterns for achieving the end results. One of the primary concerns is addressing points where bottlenecks can occur and having a solution that is flexible and can be distributed as message volumes increase.


Simple consistent hash function:

{% highlight javascript linenos %}

// 32 bit FNV-1a hash
// Ref.: http://isthe.com/chongo/tech/comp/fnv/
function fnv32a( str )
{
    var FNV1_32A_INIT = 0x811c9dc5;
    var hval = FNV1_32A_INIT;
    for ( var i = 0; i < str.length; ++i )
    {
        hval ^= str.charCodeAt(i);
        hval += (hval << 1) + (hval << 4) + (hval << 7) + (hval << 8) + (hval << 24);
    }
    return hval >>> 0;
}

{% endhighlight %}

