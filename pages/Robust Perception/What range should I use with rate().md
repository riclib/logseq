---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/what-range-should-i-use-with-rate
author: [[Brian Brazil]] 
---
> Choosing what range to use with the rate function can be a bit subtle.

# What range should I use with rate()?


Choosing what range to use with the `rate` function can be a bit subtle.

The general rule for choosing the range is that it should be at least 4x the scrape interval. This is to allow for various races, and to be resilient to a failed scrape.

Let's say you had a 10s scrape interval, and scrapes started at t=0. The rate function needs at least two samples to work, so for a query at t=10 you'd need 1x the scrape interval. At t=20, the scrape at that time may not have been fully ingested yet so 2x will cover you for two samples back to t=0. At t=29 that scrape might still not have been ingested, so you'd need ~3x to be safe. Finally you want to be resilient to a failed scrape. If the t=20 scrape fails and you're at t=39 but the t=30 scrape is still ongoing, then you'd need ~4x to see both the t=0 and t=10 samples. So a 40s rate (i.e. `rate(my_counter_total[40s]`) would be the minimum safe range. Usually you would round this up to 60s for a 1m rate.

Another consideration is that if you're using [query\_range](https://www.robustperception.io/step-and-query_range), such as in graphing, then the range should be at least the size of the step. Otherwise you'll skip over some data. Grafana's `$__rate_interval` can be useful here.

On the other hand you don't want to go too long for your ranges. If you take a rate over an hour and alert on that, that's all well and good until the underlying condition stops and then you have to wait an hour until the alert goes away. So on one hand longer ranges can make it easier to spot trends, but the averaging effect can also increase reaction times.

If you do want to have averages over different ranges (and rates are fundamentally averages), don't create recording rules for every potential range. That's wasteful, causes confusion, and can be challenging to maintain. This is as you can't compare rates over different ranges (e.g. a 5m rate and a 10m rate aren't directly comparable), and you'd have to track which is meant to be used where. As with scrape and evaluation intervals it's best to have one standard range for sanity - usually 1m, 2m, or 5m - so create one set of recording rule with a low range, and then use `avg_over_time` in graphs and alerts when you want to average over a longer period.

So to summarise, use a range that's at least 4x your scrape interval, choose one consistent range across your organisation for recording rules, and use `avg_over_time` if you need an average over a longer period for graphs and alerts.

_Not sure how to keep your recording rules maintainable? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
