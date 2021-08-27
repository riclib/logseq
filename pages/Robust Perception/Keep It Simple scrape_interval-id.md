---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/keep-it-simple-scrape_interval-id
author: [[Brian Brazil]] 
---
> How many scrape intervals should you have in a Prometheus?

# Keep It Simple scrape_interval-id


How many scrape intervals should you have in a Prometheus?

In the Prometheus configuration there are two places you can configure the scrape interval: a default in the `global` section and then per-`scrape_config` overrides.

So you could decide that normally you want to scrape everything every 10s, but there's some key servers that you would like 2s for, and other key ones that are a little slower so maybe 4s is the right setting. Then there's a really slow exporter that needs 1m, and an even slower one that needs 3m. So sounds like 2s, 4s, 10s, 1m and 3m is the way to go!

Please don't.

Having such nuanced settings is likely to cause you more hassle than the benefit it brings.

The first issue is the complexity of this configuration. Every time a new service is added you need to spend time figuring out which category it falls under, and maybe even add a new value. Everyone loves meetings to decide upon fine tuned settings!

Secondly once the data is in the Prometheus, when writing queries you usually need to know the interval of the underlying data so that you can choose the most appropriate range for your `rate()` . The more intervals you have, the more likely that you'll not get that quite right. In addition working with data with different intervals can be a little tricky, as for example `rate()`s with different ranges are not comparable.

Thirdly intervals on the order of single-digit seconds is getting into profiling territory. While a metrics-based system like Prometheus can handle that in certain cases, Prometheus is not a general profiling tool and thus not the most appropriate for the job. Profiling with Prometheus takes some care in design, and is not something you always want to mix in with your general monitoring.

So what would I suggest instead? Keep things simple. Pick one scrape interval, and stick with it. Preferably not just per Prometheus, but across your team/organisation. A value in the range of 10-60s tends to be good.

If you feel the need to get higher resolution, consider whether it would be better to get double the resolution or to instead double the amount of instrumentation you have. For some problems metrics will not suffice and you will need to incorporate logs, tracing and/or profiling into your debugging. The more instrumentation you have in your code, the easier is to debug as the additional metrics will help you narrow down and correlate issues. For the problems where resolution does matter (e.g. [microbursts](https://en.wikipedia.org/wiki/Micro-bursting_(networking))) it is not guaranteed that metrics will be able to capture the issue, whereas logs should always spot it.

Metrics are complementary to other types of monitoring and debugging tools, not a replacement. Metrics give you a good view of how things are working at the system and subsystem levels, but don't cover individual requests or instruction-level timings.

For slow exporters you have three options. The first is to cut down the amount of data that they're pulling in so that you can scrape it often enough. If it's taking so long to scrape, how much CPU are those metrics costing to produce? The second option is to increase the general scrape interval to accommodate these exporters. The third option is to have a separate scrape interval for the slow exporters, however to keep things simple you should have only one of these. This interval should not exceed 2m to avoid issues with staleness, and the caveats above around multiple intervals still apply.

When setting up your monitoring resist the urge to prematurely micro-optimise. Keeping things simple will help to avoid gnarly problems down the road, so don't bring complexity upon yourself earlier than you have to!

_Want expert advice on architecting your monitoring? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
