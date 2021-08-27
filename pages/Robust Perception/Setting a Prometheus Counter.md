---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/setting-a-prometheus-counter
author: [[Brian Brazil]] 
---
> We're often asked how to call set() on a Counter. So how do you do that?

# Setting a Prometheus Counter


We're often asked how to call `set()` on a Counter. So how do you do that?

You don't.

Counters only go up, so the only operation that makes sense is to increment them. This is not something you ever need when directly instrumenting your own code.

When users ask for this it always turns out that they're taking data from another monitoring or instrumentation system and want to get it into one of our client libraries, and ultimately into Prometheus. The good news is that there is a supported way to do this via custom collectors. This way of doing this is often called ConstMetrics, due to the name of the functions that the Go client offers for this.

The basic principle is that you create a custom collector, and then at scrape time fetch the current value of the counter and return it. In Go this looks like:

package main

import "github.com/prometheus/client\_golang/prometheus"

type MyCollector struct {
  counterDesc \*prometheus.Desc
}

func (c \*MyCollector) Describe(ch chan<- \*prometheus.Desc) {
  ch <- c.counterDesc
}

func (c \*MyCollector) Collect(ch chan<- prometheus.Metric) {
  value := 1.0 // Your code to fetch the counter value goes here.
  ch <- prometheus.MustNewConstMetric(
    c.counterDesc,
    prometheus.CounterValue,
    value,
  )
}

func NewMyCollector() \*MyCollector {
  return &MyCollector{
    counterDesc: prometheus.NewDesc("my\_counter\_total", "Help string", nil, nil),
  }
}

// To hook in the collector: prometheus.MustRegister(NewMyCollector())

Usually you'll have far more than one counter, so you would expand on this general pattern.

[Java](https://github.com/prometheus/client_java#custom-collectors) and [Python](https://github.com/prometheus/client_python/#custom-collectors) have similar style APIs. This is also how exporters are written.

_Unsure about custom collectors? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
