---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/who-wants-seconds
author: [[Brian Brazil]] 
---
# Who wants seconds?

> The Prometheus instrumentation guidelines say to use seconds, and the timing functions in client libraries follow this. Why?

---
The Prometheus [instrumentation guidelines](https://prometheus.io/docs/instrumenting/writing_exporters/#naming) say to use seconds, and the timing functions in [client](https://github.com/prometheus/client_python#summary) [libraries](https://prometheus.io/client_java/io/prometheus/client/Summary.Timer.html#observeDuration--) follow this. Why?

If you've interacted a few monitoring systems and instrumented libraries you'll have come across data exposed in a variety of units. Seconds, milliseconds, microseconds, and nanoseconds are common. The choice is often tied to what time sources are [easy](https://docs.oracle.com/javase/7/docs/api/java/lang/System.html#currentTimeMillis()) [to](https://docs.python.org/2/library/time.html#time.time) [access](https://docs.oracle.com/javase/7/docs/api/java/lang/System.html#nanoTime()) for a language, the typical latencies in your application and a limitation of only integer values being possible.

I worked on a team for many years that had a de facto standard of milliseconds. All our dashboards assumed milliseconds. Sure there was the odd kilo-millisecond on a graph, but it generally worked for the best part of a decade.

And then I had to integrate another team's monitoring which had microseconds as the standard. The dashboards just weren't designed to handle a different unit, so I had to hack it in and hope it didn't cause too much confusion in an emergency.

Now you could say the problem was the inflexible dashboard, but that's taking the wrong lesson from the story.

## Instrumentation Authors Should Not Care About Units

A developer writing instrumentation shouldn't be burdened by having to choose an appropriate unit, especially as it's not often possible. I've seen an RPC system for which different uses had typical latencies ranging from nanoseconds to months - that's 16 orders of magnitude! How do you choose a good unit for that?

The real problem is a lack of a standard both across projects and within a project. To give you an idea of how easy is is to fall into this trap, the Prometheus server itself used to export all 4 of the units listed above (after a cleanup we're down to 2 now, and hopefully 1 in the future).

The thing is that units are a convenience for humans, like time zones. And just like time zones, they should be handled purely at the display layer right before a human looks at a dashboard.

The Prometheus approach is to use base units such as seconds, and then floating point numbers to store the value. Using base units without SI-prefixes like milli or micro makes it easy for the display layer (such as Grafana) to prepend an appropriate prefix and not cause kilo-milliseconds weirdness. Floating point removes the concern of the magnitude of the number, as 64 bit floats can easily handle 10s of orders of magnitude without losing precision.

So when choosing units for instrumentation, use base units like seconds and bytes. Leave display problems to the display layer.
