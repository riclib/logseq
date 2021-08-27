---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/avoid-the-wall-of-graphs
author: [[Brian Brazil]] 
---
> Data is not the same as information.

# Avoid the Wall of Graphs


Data is not the same as information.

As your system grows, new features are added and outages analysed it's natural for your dashboards to gain new graphs over time. It's usually a slow process, however the end result is almost always the same: a dashboard overloaded with graphs that's hard to interpret for someone without deep experience of it. That experience allows them to know which graphs are not relevant and can be ignored, which a casual viewer does not have.

I call this a "Wall of Graphs". The biggest I've personally experienced had about 600 graphs on one dashboard, and I've heard of instances where others have got over 1000. There's two problems with such dashboards, firstly they tend to be slow to load and secondly and more importantly they're quite arcane to use. Even for those experienced with them, the cognitive load of knowing what is important and how to correctly interpret each graph slows them down, and may cause a misstep while investigating an outage at 3am.

I've come across more than a few groups who defend such a setup, usually on the basis that they understand what it means so that's fine. I think that this is partly due to sunk costs, partly as they value their current arcane knowledge, and partly due to a belief that quantity is valuable when it comes to graphs. Each graph added to the dashboard likely made sense at the time, such as a graph would have been useful during the most recent outage, but in aggregate they have made the actual performance of the system represented on the dashboard harder to grok.

What to do instead? First I'd suggest that if there's a dashboard used for multiple purposes or by different groups of people, to instead create a dashboard each. One dashboard that tries to serve many goals ends up serving none of them. Secondly the higher level a dashboard is, the fewer graphs it should contain. So the overview dashboard for a service or binary should only cover core aggregated metrics like request rate, errors, latency, CPU, and memory usage. If you need per-process information or more detail on some particular subsystem/feature, put it on another dashboard so you can follow a logical chain of deduction rather than trying to take in everything at once. Similarly don't duplicate machine-level metrics onto service dashboards, rather jump to a machine dashboard when you need to check those.

As dashboards become more niche, it's okay for them to have more graphs. For example the [Prometheus Benchmark dashboards](https://grafana.com/dashboards/9761) we publish are not good dashboards as overview dashboards as there's far too many graphs (30+), and they're intended for those with a good knowledge of Prometheus low-level internals. This makes sense given I created the initial version for myself when I was digging into Prometheus performance back in 2016. So they're fine for an expert, but for day-to-day use a simpler dashboard or key metric would be better. For example it might make sense to have a metric for proportion of time spent in garbage collection on an overview dashboard, but detailed garbage collection metrics for one instance can live in a different debugging dashboard that can be just a click away.

In summary, remember who the audience for your dashboards are and what their goals are. Success of a set of dashboard is not measured in how many graphs they have, but typically how quickly someone can correctly triage and debug issues - whether they're an old hand on the team, or the newest hire.

_Unsure how to improve your dashboards? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
