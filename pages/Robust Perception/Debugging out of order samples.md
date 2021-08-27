---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/debugging-out-of-order-samples
author: [[Brian Brazil]] 
---
> How do you debug and resolve the "Error on ingesting out-of-order samples" warning from Prometheus?

# Debugging out of order samples


How do you debug and resolve the "Error on ingesting out-of-order samples" warning from Prometheus?

The TSDB inside Prometheus is append only, once series has a sample for t=10 you can't ingest a sample for any earlier time such as t=9. If a scrape attempts to do so you'll get a warning like:

level=warn ts=2020-05-26T11:01:50.938Z caller=scrape.go:1203 component="scrape manager" scrape\_pool=test target=http://localhost:8000/metrics msg="Error on ingesting out-of-order samples" num\_dropped=1

Here `test` is the `job_name` of a scrape config, and `target` is the URL that is being scraped. Only a count of how many samples were affected is provided, as it could be quite spammy otherwise. If you turn on debug logging with `--log.level=debug` you can get the names of the relevant series:

level=debug ts=2020-05-26T11:01:50.938Z caller=scrape.go:1245 component="scrape manager" scrape\_pool=test target=http://localhost:8000/metrics msg="Out of order sample" series="a{foo=\\"bar\\"}"

So you've a little more information, how do you fix this? First you need to understand what can cause this. This simplest but least likely case is that a single target is exposing out of order timestamps either within a scrape (i.e. is exposing the same series multiple times, with timestamps going backwards), or across scrapes. This can be checked by refreshing the scrape endpoint a few times and seeing if things are going backwards or there's duplicate series with descending timestamps. If this is the issue, it'll need to be fixed in the code of that target. Keep in mind that targets should rarely expose timestamps.

Another possible but less likely cause is that you've manged to create two targets with identical target labels. This is not possible within one scrape config as the targets will be deduplicated, but can happen across scrape configs. This does not require that the targets expose timestamps, and if it's happening you'll notice that the generated metrics like `up` and `scrape_duration_seconds` are affected. How this then happens is the two scrape loops will independently scrape the target on their own schedule. The timestamp of ingested data is by default when the scrape stats. So say the first scrape starts at t=1, the second scrape starts at t=2, the second scrape finishes as t=3, and the first scrape finishes as t=4. When the second scrape completes `up` will be ingested with the timestamp t=2, so when the first scrape completes and subsequently tries to ingest `up` with t=1 that fails due to being out of order. This is a race condition, so may not show consistently and will often only produce logs from one of the two relevant targets. To check if this is the cause, check if any other targets in your Prometheus have the same target labels as the target referenced in the log message. To fix this ensure that all targets within a Prometheus have unique target labels, either by adding distinguishing labels or removing duplicate targets.

A related potential cause is if the system time goes backwards on the machine running Prometheus. This would show up as widespread out-of-order for all targets that existed both before the time change and afterwards, affecting both scraped series and generated series like `up`. To check if this is the cause, evaluate `timestamp(up)` for some target the Prometheus is scraping and see if it is in the future. To fix this you can either wait for the system time to catch back up, or delete the "future" data from Prometheus.

The final potential cause is the most complex one, where two distinct targets are producing the same time series. Given that by now you've ensured that you don't have targets with duplicate target labels, this means that either `honor_labels` or `metric_label_configs` are involved. `honor_labels` means that you completely trust targets to override their target labels, while `metric_label_configs` allows for more sophisticated series label munging. It that can be quite tricky for you to debug - particularly if `metric_label_configs` is in use - as given the target and time series from the log message you need to reverse engineer which other targets might end up producing series with clashing labels. If the target is one where `honor_labels` and `metric_label_configs` is not configured, then you can limit your search to targets where they are configured. On the other hand if the target in the logs is one where these features are in use, then the clash could potentially be with any other target.

The most common cause of this is federation, where two Prometheus servers in a HA setup naturally contain all of the same series with different timestamps and `honor_labels` is in use but the time series aren't getting distinguished. If this is the case the fix is simply to ensure that both of the Prometheus servers have distinct `external_labels`, which is [what you should be doing in any case](https://www.robustperception.io/high-availability-prometheus-alerting-and-notification). This could also happen if you had two pushgateways which a given batch job was pushing to both of, in which case adding a distinguishing target label which doesn't clash with any of the scraped labels should fix it. If it's not a simpler case like this, you may have to spend quite a bit of time figuring out what exactly is causing the clash. Removing `honor_labels` and overly-broad `metric_label_configs` from any of your targets which don't need them would also be wise.

The `prometheus_target_scrapes_sample_out_of_order_total` metric also tracks how often this is happening across a Prometheus. Your aim should be to ensure that this never happens, as it generally indicates a coding, architecture, or configuration flaw. All this only covers targets interacting with other targets, this problem can occur with recording and alerting rules for which you can apply similar debugging techniques.

_Having problems with scrape errors? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
