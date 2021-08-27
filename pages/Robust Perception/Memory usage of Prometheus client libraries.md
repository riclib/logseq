---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/memory-usage-of-prometheus-client-libraries
author: [[Brian Brazil]] 
---
# Memory usage of Prometheus client libraries

> A common question around Prometheus client libraries is how much RAM they'll use on a busy process. There tends to be disbelief when we say it's the same as an inactive server. Let's look deeper.

---
A common question around Prometheus [client libraries](https://prometheus.io/docs/instrumenting/clientlibs/) is how much RAM they'll use on a busy process. There tends to be disbelief when we say it's the same as an inactive server. Let's look deeper.

The simplest way to test this is a small benchmark:
```python
from prometheus_client import Counter
import resource
print("Before creating counters: ", resource.getrusage(0).ru_maxrss)

counters = []
for i in range(1000):
 counters.append(Counter("counter{0}".format(i), "help"))
print("After creating counters: ", resource.getrusage(0).ru_maxrss)

for i in range(10):
 for c in counters:
   c.inc()
print("After 10 increments each: ", resource.getrusage(0).ru_maxrss)

for i in range(1000):
 for c in counters:
   c.inc()
print("After 1000 increments each: ", resource.getrusage(0).ru_maxrss)

When run this produces for me:

('Before creating counters: ', 12792)
('After creating counters: ', 13844)
('After 10 increments each: ', 13844)
('After 1000 increments each: ', 13844)
```

So the claim that a busy server is going to use the same amount of RAM as a quiet server is shown to be true.

Why is this? Surely there's buffering going on of all the increments?

The answer is no. The counter is just a value that is updated in memory upon an increment. If you were to look at the core of what a client library does, ignoring all the concurrency handling it is simply the constant memory function:
```python
def inc(self, amount)
  self.value += amount
```
Gauges are similarly simple, and Histograms are essentially just a convenient wrapper around a set of Counters; so both Gauges and Histograms are also constant memory. The quantiles in a Summary vary by implementation, it should be bounded in client libraries but if you're worried use a quantile-less Summary (which is two Counters) or a Histogram instead.

If you're wondering how Prometheus can work off just this single value rather than a stream of buffered events, check out [How does a Prometheus Counter work?](https://www.robustperception.io/how-does-a-prometheus-counter-work/)

(With the Java client the above claim is for practical purposes correct, but not the full truth. For performance it uses a [Striped64](https://github.com/prometheus/client_java/blob/master/simpleclient/src/main/java/io/prometheus/client/Striped64.java), which grows its internal data structures when it encounters contention. However this growth is bounded based on the number of CPUs in the machine, and is thus constant memory.)
