---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus
source: https://www.robustperception.io/which-are-my-biggest-metrics
author: [[Brian Brazil]]
---
> As your Prometheus usage grows and starts to get loaded, it'd be useful to know which metrics are using the most resources so that you can re-evaluate their utility.

# Which are my biggest metrics?


As your Prometheus usage grows and starts to get loaded, it'd be useful to know which metrics are using the most resources so that you can re-evaluate their utility.

PromQL is [Prometheus](https://prometheus.io/)' query language, that allows you to perform powerful calculations across your data. It can be used not only to analyse the metrics from your individual services, but also to analyse across all the metrics inside your Prometheus server.

==To know which metrics are using the most resources it'd be good to count how many time series each has==, and then display the top 10. This can be done with an expression selecting all metrics, aggregating a count based on the metric name and returning the top 10 in the [expression browser](https://prometheus.io/docs/visualization/browser/) (make sure to use the Console view):
```
topk(10, count by (\_\_name\_\_)({\_\_name\_\_=~".+"}))
```
On the [live demo](http://demo.robustperception.io:9090/graph#%5B%7B%22range_input%22%3A%221h%22%2C%22end_input%22%3A%22%22%2C%22step_input%22%3A%22%22%2C%22stacked%22%3A%22%22%2C%22expr%22%3A%22topk(10%2C%20count%20by%20(__name__)(%7B__name__%3D~%5C%22.%2B%5C%22%7D))%22%2C%22tab%22%3A1%7D%5D) this shows that _Conway's Life_[^1] has the biggest time series:  
![](http://www.robustperception.io/wp-content/uploads/2015/12/Screenshot-161215-111025-600x572.png)

We could also aggregate it by job:

topk(10, count by (\_\_name\_\_, job)({\_\_name\_\_=~".+"}))

Or see which jobs have the most time series:

topk(10, count by (job)({\_\_name\_\_=~".+"}))

==Beware that as these queries touch all time series they are relatively expensive, so it'd be best not to hammer your Prometheus with queries of this nature==.

[^1] [[Conway’s Life in Prometheus]]