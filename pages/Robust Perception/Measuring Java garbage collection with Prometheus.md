---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/measuring-java-garbage-collection-with-prometheus
author: [[Brian Brazil]] 
---
> GC stats are one of the many metrics that the Java/JVM client library exposes.

# Measuring Java garbage collection with Prometheus


GC stats are one of the many metrics that the Java/JVM client library exposes.

Assuming that `DefaultExports.initialize();` has been invoked, the Java client will expose a number of JVM metrics out of the box including memory pools, memory allocations, buffer pools, threads, JVM version, loaded classes, and of course garbage collection.

The GC information comes from [GarbageCollectorMXBean](https://docs.oracle.com/javase/8/docs/api/java/lang/management/GarbageCollectorMXBean.html), and is exposed as the `jvm_gc_collection_seconds` summary. In particular `jvm_gc_collection_seconds_count` for the number of GCs, and `jvm_gc_collection_seconds_sum` for how long they've all taken.

These are counters, so we can take a rate:

[![](https://www.robustperception.io/wp-content/uploads/2019/02/Screenshot_2019-02-22_13-13-36.png)](https://www.robustperception.io/wp-content/uploads/2019/02/Screenshot_2019-02-22_13-13-36.png)

Here it seems `PS Scavenge` is happening once every 2 seconds or so, and `PS MarkSweek` is rare. You may ask which of those is the young generation and which the old/tenured, but this is not something the JVM exposes so you have to know which is which in your setup [given the name](https://blogs.oracle.com/jonthecollector/our-collectors).

A GC every 2 seconds sounds excessive, so let's check how long they're taking:

[![](https://www.robustperception.io/wp-content/uploads/2019/02/Screenshot-from-2019-02-22-13-18-45.png)](https://www.robustperception.io/wp-content/uploads/2019/02/Screenshot-from-2019-02-22-13-18-45.png)

So they're only taking about 1.5ms on average, which is acceptable. The single `PS MarkSweep` took 45ms, but they're rare.

Finally using `rate(jvm_gc_collection_seconds_sum[1m])` you can see what proportion of time each type of GC is taking up, which per the previous numbers is under 0.1% so not a concern at all.

_Want to learn more about optimising Java applications? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
