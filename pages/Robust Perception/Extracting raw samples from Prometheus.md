---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/extracting-raw-samples-from-prometheus
author: [[Brian Brazil]] 
---
> Sometimes you want the raw samples inside Prometheus for analysis or debugging. How do you get that?

# Extracting raw samples from Prometheus


Sometimes you want the raw samples inside Prometheus for analysis or debugging. How do you get that?

The `query_range` endpoint that you usually use is great for graphing, however it doesn't expose the raw samples. `query_range` works by evaluating PromQL at every step, so can miss samples if there're more than one sample between steps. The resulting output will also have the timestamp of the evaluation step rather than the sample.

So how can you get the raw samples? The answer is to use a range vector selector with the `query` endpoint. For example [http://demo.robustperception.io:9090/api/v1/query?query=up\[1m\]](http://demo.robustperception.io:9090/api/v1/query?query=up[1m]) will return the last minute worth of raw data for the `up` time series, from there you can look at it by hand or process it in your own scripts and code. For something a little more human readable you can use the [Console tab of the Expression Browser](http://demo.robustperception.io:9090/graph?g0.range_input=1h&g0.expr=up%5B1m%5D&g0.tab=1).

As this is raw data, you can't use PromQL to do any processing on the samples. You should also keep in mind how big the results might be, pulling in several hundred megabytes of data in this way is unlikely to end well. Finally this technique only shows normal samples, and does not include stale markers. If you want the stale markers (likely only if you're debugging staleness itself) you can access them via the remote read endpoint at `api/v1/read` which is more difficult to use but also a bit more efficient as it uses the remote read protocol.

_Want to know more about extracting data from Prometheus? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
