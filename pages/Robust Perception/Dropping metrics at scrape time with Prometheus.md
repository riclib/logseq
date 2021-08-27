---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/dropping-metrics-at-scrape-time-with-prometheus
author: [[Brian Brazil]] 
---
> It's easy to get carried away by the power of labels with Prometheus. In the extreme this can overload your Prometheus server, such as if you create a time series for each of hundreds of thousands of users. Thankfully there's a way to deal with this without having to turn off monitoring or deploy a new version of your code.

# Dropping metrics at scrape time with Prometheus


It's easy to get carried away by the power of labels with Prometheus. In the extreme this can overload your [Prometheus](https://prometheus.io/) server, such as if you create a time series for each of hundreds of thousands of users. Thankfully there's a way to deal with this without having to turn off monitoring or deploy a new version of your code.

Firstly you need to find which metric is the problem. Go to the expression browser on Prometheus (that's the /graph endpoint) and evaluate  `topk(20, count by (__name__, job)({__name__=~".+"}))`. This will return the 20 biggest time series by metric name and job, which one is the problem should be obvious.

Now that you know the name of the metric and the job it's part of, you can modify the job's scrape config to drop it. Let's say it's a metric called `my_too_large_metric`. Add a `metric_relabel_configs` section to drop it:

scrape\_configs:
 - job\_name: 'my\_job'
   static\_configs:
     - targets:
       - my\_target:1234
   metric\_relabel\_configs:
   - source\_labels: \[ \_\_name\_\_ \]
     regex: 'my\_too\_large\_metric'
     action: drop

All the samples are still pulled from the job being scraped, so this should only be a temporary solution until you can push a fixed version of your code.

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
