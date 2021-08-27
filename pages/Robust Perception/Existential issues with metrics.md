---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/existential-issues-with-metrics
author: [[Brian Brazil]] 
---
# Existential issues with metrics

> The Prometheus instrumentation best practices say to "Avoid missing metrics". Let's look at why, and how to deal with it.

---
The Prometheus instrumentation best practices say to ["Avoid missing metrics"](https://prometheus.io/docs/practices/instrumentation/#avoid-missing-metrics). Let's look at why, and how to deal with it.

Prometheus is a metric-based time series monitoring system. A key aspect of this style of monitoring is that time series have a continuous existence over a period of time, and that scrapes are a snapshot of what those time series look like at regular intervals. We might miss some data due to an application starting/stopping/crashing between scrapes, but `rate()` will get the right result on average due to its extrapolation logic.

On average doesn't mean that it's going to catch every single increment in all cases, it means that if you have lots of increments it'll produce an accurate answer. In general a metrics-based monitoring system is not suitable for anything that cares about individual increments, you would want an event-logging system like the ELK stack for that.

Where this usually comes up is for metrics with labels. It is recommended to initialise metrics to zero when your application starts, so that when there's an increment on a subsequent scrape we'll be able to see that the value has increased over time.

If the counter just appears with the value 1 due to being initialised and incremented in the same instant, we can't distinguish between that being a brand new counter that was just incremented, or a counter that was incremented years ago that Prometheus is only successfully scraping now for whatever reason (brand new Prometheus, recently returned from service discovery, target was inaccessible for a while). Put another way `rate()` looks for an increase over time; neither a single sample with the value 1 nor a series with samples that are all 1 are an increase.

Another issue is that if you are doing math across multiple metrics and some of them don't always exist, this can be challenging to deal with and lead to alerts not firing. An example of this is if you have a failure metric and a success metric (ignoring that it's recommended to have a failure and total metric instead). If these were labeled and there were no successes, then calculating the failure ratio as `rate(failure[1m]) / (rate(failure[1m]) + rate(success[1m]))` would return an empty result due to the lack of a matching labelset in the success metric. So a 100% failure ratio would cause an alert on a high level of failures to never fire!

So how do you initialise counters and other metrics to zero? If they have no labels, this happens automatically. If there are labels you can call `labels()`/`WithLabelValues()` (depending on which language you're using), but without the `inc()`/`set()`/`observe()`. For example in Java you could do:
```
  static final Counter counter = Counter.build()
     .name("my_counter_total").help("Total requests.")
     .labelNames("foo").register();
  static {
     counter.labels("bar");
     counter.labels("baz");
  }
```
to initialise the bar and baz labelsets. The previous article on [Label Lookups and the Child](https://www.robustperception.io/label-lookups-and-the-child/) also had an example of a common pattern where a class instance was tied to a single label value, which was initialised in the constructor.

If you're doing things more dynamically, as may happen in the success/failure example above, what you might do is store all the Child objects you could need as local variables soon as you know the labels. That way even if there's no failures or successes, you know both metrics will exist.

What happens if it's not possible to initialise the metrics in advance at application startup?

For the case where a counter is being initialised and immediately incremented, users usually care about detecting that this happened more than the exact value (edge triggering on single events being unwise in a metrics-based system is a topic for a different day). To detect any increase we can do:
```
  # We saw a normal increase.
  rate(my_counter_total[5m]) > 0   
or 
  (  # If the value is non-zero, the counter didn't exist a minute ago, 
     # and the target was up a minute ago.
     (my_counter_total != 0 unless my_counter_total offset 1m) 
   and 
     (up offset 1m == 1)
  )
```
For the case where you're working with a metric that may not exist, such as the success/failure example, the approach is to `or` in the missing labelsets based on some metric we know exists - commonly `up`. So for example the failure ratio from above would become:
```
(rate(failure[1m]) or up * 0)
  /
((rate(failure[1m]) or up * 0) + (rate(success[1m] or up * 0))
```
These expressions aren't exactly trivial, which is why it's advised to do initialisation at startup.
