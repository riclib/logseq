---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/failing-a-scrape-with-the-prometheus-go-client
author: [[Brian Brazil]] 
---
> As previously mentioned partial failure is hard to deal with.

# Failing a scrape with the Prometheus Go client


As [previously mentioned](https://www.robustperception.io/remote-read-and-partial-failures) partial failure is hard to deal with.

Usually when an exporter is scraped the communication with whatever application it is talking to all goes fine, and the scrape is successful. What happens though if one command or RPC fails? The [exporter guidelines](https://prometheus.io/docs/instrumenting/writing_exporters/#failed-scrapes) generally recommend returning a HTTP 500 in that case to fail the scrape, as the user should already have some form of alerting in place to handle `up` being 0. In addition both partial and total failure should be rare, so there's usually not much point in putting the burden of more nuanced failure handling on to your users.

How do you actually make that 500 happen though? With the Java or Python clients you can throw an exception in the relevant code. With the Go client, there's a little bit more to it. There's a utility function called `NewInvalidMetric` which will cause the scrape to fail, for example:

func (c collector) Collect(ch chan<- prometheus.Metric) {
  data, err := SomeFunction()
  if err != nil {
    ch <- prometheus.NewInvalidMetric(
          prometheus.NewDesc("myexporter\_error",
            "Error running SomeFunction", nil, nil),
        err)
  }
  # Rest of Collect code goes here.
  return
}

This will produce a HTTP 500 whose body will look something like:

An error has occurred while serving metrics:

error collecting metric Desc{fqName: "myexporter\_error", help: "Error running SomeFunction", constLabels: {}, variableLabels: \[\]}: contents of the err

As you can see the metric help can provide some human readable information, and the `err` is also rendered. Note that for `NewInvalidMetric` to work, the `err` must not be `nil`.

_Have a question about instrumentation? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
