---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/looking-beyond-retention
author: [[Brian Brazil]] 
---
> How can you view older data, while keeping your monitoring reliable?

# Looking beyond retention


How can you view older data, while keeping your monitoring reliable?

Prometheus is designed around the notion of reliability of monitoring, it only needs local disk and network access to work. There's [no complex clustering or distributed systems](https://www.robustperception.io/monitoring-without-consensus), even federation is designed with the idea that it's a normal scrape so it's easy to reason about failure in the same way as for any other target. No matter what weird stuff goes on you want your alerting, and subsequent debugging via dashboards, to keep on working.

This reliability has a disadvantage though, if you're limited to local disk that means you can only store as much data as you can fit on to one machine. For example a 1TB disk can hold roughly two months of data for a Prometheus ingesting 100k samples/s. If you want to seamlessly access historical data when you need to you, such as for annual capacity planning, this presents a challenge as going beyond one machine's storage means a clustered system of some form is required.

This is where remote write and remote read come in. They're a way for Prometheus to sends its metrics out to such a clustered system and read them back in again, but designed so that if the other end is down then then your alerting and graphing queries will continue to work off local data. The outcome is that during an outage you'll still have all your recent data on your dashboards, which is sufficient for the vast majority of debugging. The query API will return a warning in this case, though the missing historical data is going to be pretty obvious. Alerts on months old historical data tend not to be the most critical, them being delayed a bit until the clustered system is all working again is not the end of the world (I'd aim for my critical alerts to not need more than an hour of recent data).

The way things work is that when data is sent out on remote write the external labels are attached. When reading back in via remote read, Prometheus adds the external labels to your selectors so you'll get back the series that that particular Prometheus wrote, and those external labels are then stripped before being used by PromQL so they'll match the original series.

While a Prometheus reading back its own historical data is the main intended use case, you can also access other data. If a PromQL selector explicitly specifies a matcher for an a label name which is in external labels, then the above processing doesn't happen for that label name for the remote read. One thing to watch out for is that Prometheus is smart enough to not send remote read requests for time ranges covered by local storage, avoiding unnecessary network traffic. This optimisation can be disabled by enabling the `read_recent` option.

So that's the idea behind Prometheus remote write and read. Local storage is always there with reliable recent data - it doesn't make sense to have a Prometheus without it. And remote clustered storage systems can be transparently accessed for historical data, but any failures are isolated to prevent taking out your monitoring when there's an issue. Even if you're primarily using something like Thanos or Cortex, the local Prometheus is always there as a backup for emergencies.

_Want reliable monitoring? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
