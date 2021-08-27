---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/measuring-the-performance-impact-of-meltdownspectre-with-promtheus
author: [[Brian Brazil]] 
---
> The world of infosec is alarmed right now over the recent security vulnerabilities disclosed by Google on Wednesday that affect Intel, AMD, and ARM chips.
The now infamous Meltdown and Spectre bugs allow for the reading of sensitive information from a system's memory, including passwords, private keys and other sensitive information.

# Measuring the performance impact of Meltdown/Spectre with Prometheus


The world of infosec is alarmed right now over the recent security vulnerabilities [disclosed by Google](https://security.googleblog.com/2018/01/todays-cpu-vulnerability-what-you-need.html) on Wednesday that affect Intel, AMD, and ARM chips.  
The now infamous Meltdown and Spectre bugs allow for the reading of sensitive information from a system's memory, including passwords, private keys and other sensitive information.

Thankfully fixes are being swiftly rolled out to patch these issues, however they come at a performance cost which we will use Prometheus to explore in this blogpost.

In order to test performance degradation, I tested a [patched](http://kernel.ubuntu.com/~kernel-ppa/mainline/v4.14.11/) and unpatched Ubuntu VM, using [Prometheus](https://prometheus.io/) to monitor the `process_cpu_seconds_total` and `requests_total` sent to a [simple Go application](https://github.com/RobustPerception/go_examples/tree/master/meltdown_spectre_test) instrumented with the [Prometheus client\_golang library](https://github.com/prometheus/client_golang).

The `process_cpu_seconds_total` metric comes automatically from the Go client and measures the total CPU usage by the process. In the [example code,](https://github.com/RobustPerception/go_examples/blob/master/meltdown_spectre_test/main.go) `requests_total` was added as a counter, and measures the total number of requests received by the application.

In order to inject load, I used [Apache Bench](https://httpd.apache.org/docs/2.4/programs/ab.html) to send 10,000 requests to the application. Finally in order to see the difference I divided the per-second average rate of increase of each time series ([rate](https://prometheus.io/docs/prometheus/latest/querying/functions/#rate())).

  rate(process\_cpu\_seconds\_total\[1m\])
/
  rate(requests\_total\[1m\])

This gives the average CPU usage per query, which is on the order of 200 microseconds for this trivial example.[![](https://www.robustperception.io/wp-content/uploads/2018/01/Screen-Shot-2018-01-05-at-16.22.56-1.png)](https://www.robustperception.io/wp-content/uploads/2018/01/Screen-Shot-2018-01-05-at-16.22.56-1.png)

The above graph shows a ~5% increase in time spent on the CPU when the load is injected between the unpatched and patched Ubuntu servers.

This is likely due to the changing of "speculative execution" which is the cause of security flaw but is also an optimisation technique and fundamental part of how modern CPUs operate.

_Want to regain some of that performance loss? [Contact us](mailto:prometheus@robustperception.io)._

Published by Conor Broderick in [Posts](https://www.robustperception.io/category/posts)
