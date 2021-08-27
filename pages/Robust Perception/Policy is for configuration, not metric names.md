---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus, monitoring, time series
source: https://www.robustperception.io/policy-is-for-configuration-not-metric-names
author: [[Brian Brazil]]
---

-
  > Metric names are part of a time series's identity so shouldn't include information unrelated to identity
- I previously looked at[^1] some of the consideration for choosing target labels. Similar applies to the ==names of metrics, you want something descriptive but not longer than it needs to be== . In this context, I'd like to talk about how to control which metrics you subsequently want sent out via federation or remote write.
- ==It may be tempting to add `remotewrite:` or `federate:`== to a metric names that are produced from recording rules and then filtering on that with relabelling or selectors respectively, however it comes with pitfalls.
- The first is that ==you'd have hardcoded policy into a metric name==, which isn't innately tied to that metric's identity. If the policy about what gets sent changes, then so does the metric name - which will cause breakage for everything depending on it. A metric's identity can be changed by doing math such as aggregation on it in PromQL, but neither remote write nor federation do math so can't change the meaning of the numbers.
- The second is that ==it would cause confusion with the standard naming scheme for recording rules==[^2], as these names would incorrectly indicate that that metric was an aggregation split out by a `remotewrite` or `federate` label. This would make it harder to tell if PromQL rules are correct, as you can no longer rely on the name to indicate the relevant labels in play. ==Put another way it adds noise to the metric name== that doesn't a reader help you understand what it means, instead adding what is an irrelevant implementation detail about your monitoring architecture from a PromQL standpoint.
- ==It's no better to add a label such as `federate="true"` as part of your recording rules==. Once again if policy changes everything downstream could break, and having unexpected labels usually requires adjustments to your PromQL. One impact of that is that you'd find it harder to reuse generic alerts and dashboards.
- So how should you handle this? I'd say keep it simple, ==don't pick which recording rules you want sent out via federation or remote write on a case-by-case basis. Instead do so in broad strokes==, such as `{__name__=~"job:.*}` to pick up all recording rules that are aggregated up to the job level or `{__name__=~".*:.*",__name__!~".*instance.*:"}` for everything except that which has an instance label. These are selectors for federation, the same idea can be applied for the relabel actions that'd be used for remote write. You could also have more granular metric name selection in your regexes, but ==by the time that instance label is aggregated away the cardinality should generally be low enough that it's not generally worth micro-optimising resource costs== when compared against the maintenance costs of such a configuration.
- [^1] [[Target labels are for life, not just for Christmas]]
- [^2] [[Recording rules - Prometheus]]