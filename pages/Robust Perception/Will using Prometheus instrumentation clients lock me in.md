---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/will-using-prometheus-instrumentation-clients-lock-me-in
author: [[Brian Brazil]] 
---
> Considering using Prometheus, but worried about committing to using our clients?

# Will using Prometheus instrumentation clients lock me in?


Considering using Prometheus, but worried about committing to using our clients?

When starting out with a new technology it is wise not to end up in a situation later on where you've decided not to use it, but it's still lingering around your codebase. This is one of the reasons I recommend to start out with the Node exporter when exploring Prometheus, rather than going all-out with instrumenting your code base. When it comes to providing instrumentation from inside for Prometheus from inside your applications, this concern should be kept in mind but should not be the only factor in your decision.

The first thing to be aware of is that the Prometheus text format used for transferring metrics has become something of a defacto standard with many different commercial and non-commercial monitoring systems able to ingest it, including [Datadog](https://www.datadoghq.com/blog/monitor-prometheus-metrics/), [InfluxDB](https://www.influxdata.com/integration/prometheus-monitoring-tool/), [Outlyer](https://www.outlyer.com/features/), [Sensu](https://blog.sensuapp.org/the-sensu-prometheus-collector-972c441d45e), [Metricbeat](https://www.elastic.co/guide/en/beats/metricbeat/6.1/metricbeat-metricset-prometheus-collector.html), [Sysdig](https://sysdig.com/blog/prometheus-metrics/), and [SignalFX](https://signalfx.com/blog/metrics-from-prometheus-exporters-are-now-available-with-the-sfx-smart-agent/). By using a Prometheus client you not only get modern instrumentation that is designed to work with the full Prometheus data model, but offers you options in the future if you decide to use other monitoring systems. Furthermore there's [work underway](http://openmetrics.io/) to make the Prometheus text format an actual standard, and the Prometheus client libraries will almost certainly form part of that.

The second thing is that Prometheus clients are not limited to outputting the Prometheus text format. That is how they are usually used, but the code that does so uses public APIs that you can use to output to other formats. As an example of this the [Python](https://www.robustperception.io/exporting-to-graphite-with-the-prometheus-python-client/), [Java](https://www.robustperception.io/exporting-to-graphite-with-the-prometheus-java-client/), and Go client libraries can all export to Graphite out of the box. The Prometheus clients are intended to be open.

So if you're using Prometheus, don't be afraid to get the most out of your instrumentation by using a Prometheus client library!

_Looking to integrate Prometheus with other monitoring systems? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
