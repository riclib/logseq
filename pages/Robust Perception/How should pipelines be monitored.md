---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/how-should-pipelines-be-monitored
author: [[Brian Brazil]] 
---
> For online serving systems it's fairly well known that you should look for request rate, errors and duration. What about offline processing pipelines though?

# How should pipelines be monitored?


For online serving systems it's fairly well known that you should look for request rate, errors and duration. What about offline processing pipelines though?

For a typical web application, high latency or error rates are the sort of thing you want to wake someone up about as they usually negatively affect the end-user's experience. Request rate isn't something to alert on in and of itself, however it's important to know as it's often related to errors/latency plus you'll want it for capacity planning.

A offline processing pipeline typically involves queues (such as Kafka) between various stages of computation. There's no end user eagerly waiting for a web page to load, however how long it takes for data to get through is a key metric. Similarly if data goes in but an error causes it to be dropped or otherwise not correctly processed that's usually something to be concerned about. In addition there's how much data is sitting in each queue, how fast data is being added, and how fast data is being removed.

Many will have alerts on too much data being in a queue, and this tends to be a bit spammy. First off any alert on a fixed threshold tends to get out of date as traffic grows, in the same way it's better to alert on the ratio of HTTP errors to total requests rather than how many happen per second. The more serious issue however is that one queue having a certain number of items in it doesn't mean that the overall pipeline is processing data too slowly, and setting thresholds to avoid such false positives would miss actual problems. This is typical for alerts that work off causes rather than symptoms.

What's a better approach then? What we could do is alert on what we really care about, how long data is taking it to get through the pipeline. For simplicity, let's presume a simple single-sharded FIFO setup. If you look at a batch of data when it comes out of the end pipeline and compare that to when it was inserted into the start of the pipeline, you'll then know how long the pipeline is taking to process data - which is what we care about and can alert on. One approach for this is that each item to be processes has its initial insertion time added as metadata, which is preserved as it winds its way through the pipeline. If there's continuous data then this is sufficient, on the other hand if there's lulls when there's nothing to do then adding heartbeats in the form of dummy items to be processed helps.

This works fine for relatively quick pipelines, but say that you have a pipeline that takes a significant amount of time to normally process things where you don't have much slack in your SLA (and can't get some added). If there was a bottleneck early in the pipeline, it could take too long for alerting on overall processing time to catch it. In this case alerting on the size of queues can be appropriate. I would suggest alerting not just on the value being too high, but also that it is increasing. In PromQL terms that's `deriv(my_queue_size[30m]) > 0`, which avoids being woken up to find that the problem is already fixing itself.

_Have questions about reducing alert noise? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
