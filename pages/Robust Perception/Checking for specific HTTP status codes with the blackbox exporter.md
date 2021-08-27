---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/checking-for-specific-http-status-codes-with-the-blackbox-exporter
author: [[Brian Brazil]] 
---
> How would you check that a HTTP endpoint is returning a 204?

# Checking for specific HTTP status codes with the blackbox exporter


How would you check that a HTTP endpoint is returning a 204?

We've previously looked at using the [blackbox exporter to check that a 2xx response code](https://www.robustperception.io/checking-for-http-200s-with-the-blackbox-exporter) is being returned, you might however consider only particular 2xx codes as okay.

Your first thought might be to use an alert based on the `probe_http_status_code` metric, however this has several downsides. Firstly as a gauge if there's a [failed scrape then the alert will not fire](https://www.robustperception.io/alerting-on-gauges-in-prometheus-2-0). Attempting to correctly handle that, particularly if there's multiple acceptable status codes, would require some involved PromQL. Secondly you now would need several alerts to catch scrape failure, probe failure, and then the status code being incorrect.

There is a much simpler and easier way. `probe_success` is intended to be all you need to capture whether or not your probe succeeded, whereas metrics like `probe_http_status_code` are intended for after the fact debugging to help figure out why a probe failed (e.g. was it a 404 or a 500?). Accordingly you can configure a module in your `blackbox.yml` with what you consider to be success, for example:

modules:
  http\_204:
    prober: http
    http:
 valid\_status\_codes: \[204\]

You can then use the usual `up{job="blackbox"} == 0 or probe_success{job="blackbox"} == 0` alerting expression for all your probes.

This applies to other metrics too, such as whether TLS was used, the protocol version, whether there were redirects, and how long the probe took.

Defining what success is in the blackbox exporter is simpler and more reliable than trying to replicate that in PromQL.

_Have questions about blackbox monitoring? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
