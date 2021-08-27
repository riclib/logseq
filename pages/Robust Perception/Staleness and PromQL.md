---
created: [[Aug 4th, 2021]]
type: #clipping
status: #reviewed
tags: prometheus 
source: https://www.robustperception.io/staleness-and-promql
author: [[Brian Brazil]] 
---
> How should a monitoring system deal with metrics no longer being there?

# Staleness and PromQL
How should a monitoring system deal with metrics no longer being there?
- One of the advantages of pull-based monitoring is that you can tell when a scrape fails, as against some data not appearing for what could be a number of reasons. Even with that information, how should it be exposed semantically to PromQL? PromQL allows evaluation at any time, so we can't just see what's stale "now".
  
  As of Prometheus 2.0, the way this is handled is via "staleness markers". These are special samples (internally implemented as a [special type of NaN](https://github.com/prometheus/prometheus/blob/19152a45d8a8f841206d321f79a60ab6d365a98f/pkg/value/value.go#L28), so you may see them referred to as "stale NaNs") which are not exposed to users and that indicate that a time series has gone stale. Like all samples they have timestamps.
  
  Let's take an example. A sample is exposed for a time series on one scrape, but not the next, and then exposed again. The time series will now have the values say `t0=7 t10=stale t20=9`.  If an instant vector selector is evaluated at `t=0` up to just before `t=10`, then 7 will be returned. If an instant vector selector is evaluated at `t=10` or up to just before `t=20` then that time series will not be returned. From `t=20` onwards, 9 will be returned. In this way time series the appear and disappear are correctly handled (even though they remain something to avoid).
  
  What about range vector selectors? Range vectors completely ignore stale markers, so functions like `avg_over_time` and `rate` will act only on the non-stale samples. `count_over_time` over the above data would return 2 for example.
  
  ==If a scrape fails, for whatever reason, then all time series from the previous scrape will be marked as stale==. This avoids aggregations double counting when a target has failed, a new target has been brought up to replace it, but the original target is still in service discovery. This does however mean you need to be _a little careful_ [^1] when alerting on instant vectors with Prometheus 2.x to ensure one failed scrape doesn't break your alerting.
  
  Things are trickier when a target disappears from service discovery. It's possible that the target will reappear before the next scrape would have been scheduled, so we can't just write out stale markers for when the next scrape would have been when a target goes away. ==What we do instead is wait until after such a recreated target would have ingested data (which is about 2 scrape intervals), and then ingest those stale markers==. If the target was recreated then these stale markers will be rejected as the TSDB only allows appending data - not replacing samples at the latest or previous timestamps.
  
  This behaviour for targets going away depends on Prometheus continuing to run for a bit after the target disappears so the delayed stale markers can be ingested. If Prometheus were to be restarted or crash, then they'll be missing. In this case the behaviour falls back to the pre-2.0 logic of considering a series stale if there's no samples within the 5 minutes before the evaluation time. This logic isn't perfect (which is why there's stale markers now), however it's sufficient for the odd time Prometheus restarts. The 5 minutes applies generally, as PromQL only looks back 5 minutes to find samples. In the earlier example, 9 would only be returned for evaluations up to `t=320`. One result of this is that scrape and evaluation intervals larger than about 2 minutes are to be avoided, as a single failed scrape would run into this.
  
  The staleness logic isn't trivial, however it by and large Just Works without you having to worry about it.
	- [^1] [[Alerting on gauges in Prometheus 2/0]]