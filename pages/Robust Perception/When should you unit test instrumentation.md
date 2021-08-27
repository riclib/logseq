---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/when-should-you-unit-test-instrumentation
author: [[Brian Brazil]] 
---
# When should you unit test instrumentation?

> Should you unit test every bit of instrumentation you add? Not always.

---
Should you unit test every bit of instrumentation you add? Not always.

I've noticed that some are fanatic about unit testing everything, including all metric instrumentation. While unit testing is generally a good idea, mandating it is not always a win.

For example if you're writing an RPC library that had request logs as a feature, those logs would definitely be something to unit test. Similarly metrics for the number of requests, their latency and failure rates should be unit tested. These metrics are features in and of themselves, and likely heavily depended on by users.

What I would advise caution in always unit testing are the equivalent of debug or application logs. More obscure metrics that are only of interest to the developer and others deep into the system are less valuable to unit test, just as unit testing `log.Debug("Using experimental RPC compression")` is probably not a good tradeoff.

Why do I say this? Well if your policy is that everything must be unit tested, then fewer debug metrics will be created. Fewer debug metrics means that debugging these code paths in live systems will be harder, requiring you to add the metric that would have been useful, rollout the new code, and wait for the problem to reoccur.

Ah, but doesn't the lack of unit tests increase the risk of these metrics being broken? This is definitely true, it's my experience that about 5% of all metrics will be broken in some way that makes them not useful; requiring a fix and once again waiting for the code to rollout and the problem to reoccur. It's my opinion that this is still a win though. Consider 20 metrics that a developer may wish to add on a whim. It's better to get 19 of those that are useful, than only 3-4 useful metrics being created due to the hassle of having to unit test every metric.

I propose the following rule of thumb: Unit test metric instrumentation if you'd unit test a log message with the same information.

Next week I'll look at [how to unit test instrumentation](https://www.robustperception.io/how-to-unit-test-prometheus-instrumentation/).
