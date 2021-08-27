---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/on-the-naming-of-things
author: [[Brian Brazil]] 
---
# On the naming of things

> How you choose to name metrics is important. If everyone choose different schemes it'd lead to confusion, irritation and prevent us from sharing and reusing each others' work. I'd like to share some guidelines to help keep things sane for everyone.

---
How you choose to name metrics is important. If everyone choose different schemes it'd lead to confusion, irritation and prevent us from sharing and reusing each others' work. I'd like to share some guidelines to help keep things sane for everyone.

## Suffixes

In Prometheus there's a few special suffixes for time series: `_total`, `_count`, `_sum` and `_bucket`. Of these you should always use `_total` with [Counters](http://www.robustperception.io/how-does-a-prometheus-counter-work/), and otherwise avoid using these suffixes in your metrics. A Gauge ending with `_count` or `_total` is quite confusing, as it implies it's a Counter rather than a Gauge.

## Units

Working back, the next thing at end of a metric name is a unit. You should always specify the units you're working with for clarity, and where possible use or convert to base units such as seconds and bytes. This is to avoid a confusing mix of seconds, milliseconds, microseconds and nanoseconds all coming from one binary (at one point Prometheus itself exposed all of these, it was quite irritating).

Units should be plural, e.g. seconds, requests, items. For seconds we usually go with `_seconds_` rather than `_s_` as it's less ambiguous.

## Namespacing

A metric named `http_requests_total` is close to useless. Is that HTTP requests as they enter the HTTP code, at the auth middleware, in the RPC subsystem or when the hit your code? That's key information when interpreting the metric.

`http_server_requests_total`, `auth_http_requests_total`, `rpc_server_http_requests_total`, and `businesslogic_requests_total` would be some better options. Keep in mind that while these all measure HTTP requests, as they're at different parts of the call stack it'd be incorrect to combine these into one metric.

## Other things to watch for

Don't put the names of the labels in the metric name, such as `http_server_requests_by_method_total`. When the label is aggregated away, this will cause confusion. There's one very rare case where this is okay, and that's where you need to distinguish multiple metrics based on the same events but which have different labels for performance reasons.

Similarly don't put the type of the metric in the name, such as `gauge`, `counter`, `summary` or `map`. This is already implied by the existing naming scheme, adds no information and will cause confusion as you process and write rules based on the metrics. For example whether you calculated request rate from a Counter or Summary is irrelevant.

Prometheus favours snake case, avoid camelCase when creating your own metrics.

Don't use colons in metric names. These are for the use of a person writing [recording rules](https://prometheus.io/docs/practices/rules/) on the Prometheus server.

And finally if you find yourself procedurally generating metric names when doing direct instrumentation, you should probably be using a label instead.

If you follow these guidelines not only will your users thank you for following the standard, but you'll find it easier to use other people's metrics too!

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
