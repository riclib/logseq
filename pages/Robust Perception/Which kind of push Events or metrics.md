---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/which-kind-of-push-events-or-metrics
author: [[Brian Brazil]] 
---
# Which kind of push? Events or metrics?

> Continuing in our exploration of the ongoing epic saga of push vs. pull where the very future of humanity is at stake, let's look at two general classes of push that are often conflated.

---
Continuing in our exploration of the ongoing epic saga of push vs. pull where the very future of humanity is at stake, let's look at two general classes of push that are often conflated.

An important question when looking at a push-based monitoring system is what exactly it is that you are pushing.

You could be pushing individual events. [StatsD](https://github.com/etsy/statsd/wiki) is the canonical example of this. The way this works is that the process of interest will have an event (such as a HTTP request) and send a message saying something along the lines "please increment the number of HTTP requests by 1, and record that a HTTP request took 150ms" to a collector. There might be an additional breakdown, such as that it was a GET/POST or which path it hit, but the key point is that every single event results in a message to the collector. The collector then aggregates these across time into metrics, and usually pushes on these metrics to somewhere else.

You could be pushing metrics. [Graphite](https://graphite.readthedocs.io/en/latest/) is a good example of this. Here the individual events are gone, as they've already been aggregated across time (such as by StatsD, or preferably in-process). Instead we send a message saying "there were 20 http requests in the past minute, and they took 97ms on average". These messages are sent regularly, such as every 10 seconds or every minute.

The more interesting aspects are around the practical issues relating to pushing events.

The primary issue with pushing events is that the volume of data is proportional to the amount of processing your application is doing. If you have twice the requests, you're going to have twice the events to handle. As you're communicating over the network to the process collecting these events (and often across machines), this can get problematic in terms of both CPU usage and network traffic.

Pushing events is often done via UDP, which is unreliable and it is expected you'll lose a few packets. If this loss was unbiased, this would be workable. However as you're sending more packets the more requests your application is processing, data loss is going to be correlated with when your application is busy rather than the uniform random behaviour you'd want. This results in biased reporting. There are optimisations you can do like combining multiple events into one packet, but even with this the sheer volume of data has lead to approaches such as throwing away 90% of data and [only sending every 10th event to StatsD](http://statsd.readthedocs.io/en/latest/types.html#counters).

To be clear there'd also be issues if you used TCP. When there's too many events to process you must either have enough RAM to queue them up, or drop some events on the floor like UDP does. The data volume is the fundamental issue here, rather than the exact implementation.

Metrics by contrast have the same resource usage to push them no matter how busy your system is. Instrumentation libraries can collect events and aggregate them into metrics inside the application process, which avoids hitting the kernel network stack for every individual event. For a well implemented instrumentation library (such as Dropwizard or Prometheus's own libraries), an event cost of only [10-20 nanoseconds](https://github.com/prometheus/client_java/tree/master/benchmark) is typical.

So when talking about push be clear whether you're talking about pushing events or metrics. Transferring metrics rather than events is a more scalable design, allowing for many more metrics to be added to your application.

As an aside, part of the reason Prometheus was created was due to the above challenges with scaling StatsD. Thus part of the Prometheus design is that it transfers metrics rather than events.
