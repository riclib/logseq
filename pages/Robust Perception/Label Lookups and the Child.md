---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus javascript
source: https://www.robustperception.io/label-lookups-and-the-child
author: [[Brian Brazil]] 
---
# Label Lookups and the Child

> The Prometheus client library guidelines recommend having a Child be returned via labels(). Why?

---
The Prometheus [client library guidelines](https://prometheus.io/docs/instrumenting/writing_clientlibs/#labels) recommend having a `Child` be returned via `labels()`. Why?

A common misstep made by those implementing client libraries for Prometheus is to have usage for labels looking something like:

```javascript
MY_COUNTER = prometheus_client.Counter('my_counter_total',
    'help', ['labelname'])
MY_COUNTER.inc(['labelvalue'], 2.0)      # Don't do this.
```

Whereas the correct pattern looks something like:
```javascript
MY_COUNTER = prometheus_client.Counter('my_counter_total',
    'help', ['labelname'])
MY_COUNTER.labels('labelvalue').inc(2.0)
```

Why is this the recommended way?

The answer is all in the label lookup. Taking the Java client (which uses the efficient [ConcurrentHashMap](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/ConcurrentHashMap.html)) as an example, looking up the object to be incremented takes 30ns uncontended while the increment itself takes 12ns. Under high contention with 4 threads it's even worse, with the lookup taking 102ns with only 18ns for the increment!

This matters If you have instrumentation that is being called in aggregate hundreds of thousands of times a second, as it can cost a non-negligible amount of CPU time. Keep in mind it's not unusual for a single client request to result in a hundred instrumentation calls, spread across a myriad of libraries.

What can we do to avoid this cost?

It turns out that for many uses of labels, the label value is constant for a given class instance. So if the result of the label lookup can be cached directly in a variable by the instrumented code, then it is a cheap memory read to find the object that needs incrementing. Thus we have `labels()` which does the lookup, the result of which is the `Child` object which can be cached and subsequently manipulated.

Let's take an example. Say we had a very simple pub-sub client which can publish to multiple queues, and each instance handled talking to one queue:
```javascript

PUBLISHED = prometheus_client.Counter('pubsub_client_published_messages_total', 
    'help', ['queue'])

class Publisher(object):
  def __init__(self, queue):
    self.__published = PUBLISHED.labels(queue)
    # Other code here.

  def publish(self, message):
    self.__published.inc()
    # Other code here.
```
This pattern has the added bonus that the `labels()` call will initialise the Child to 0, so it'll be returned on the `/metrics` even if `publish` is never invoked. This [avoids missing time series](https://prometheus.io/docs/practices/instrumentation/#avoid-missing-metrics), which are tricky to deal with and getting it wrong can and has resulted in prolonged outages.

Three final points.

The first is that this same argument applies to metrics themselves. Never be tempted when using a client library to build up a map from metric names to metric objects; that's a waste of RAM, CPU and code. Rather create the metric and keep a pointer to it in a file/class level variable for cheap access. Similarly when writing a client library, don't take in a metric name as an argument to `inc()` or other metric methods.

The second is that for metrics without labels, the client library should take care of caching the Child and initialising it to 0 for you - so you don't have to worry about any of this.

The final point is there's one additional disadvantage of `inc(['labelvalue'], 2.0)` - it gets in the way of usage for metrics without labels. If you want to pass a value to the function you'd also have to pass in an empty list of labels, which is a bit ugly. As most metrics don't have labels, this would be annoying for the person using the client library. In the worst case it may encourage labels where it'd be [better not to have them](https://prometheus.io/docs/practices/instrumentation/#do-not-overuse-labels).
