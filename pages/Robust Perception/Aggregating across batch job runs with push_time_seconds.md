---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/aggregating-across-batch-job-runs-with-push_time_seconds
author: [[Brian Brazil]] 
---
> For counting how many times a thing has happened you can use a counter and rate(), but that doesn't work across batch jobs.

# Aggregating across batch job runs with push_time_seconds


For counting how many times a thing has happened you can use a counter and `rate()`, but that doesn't work across batch jobs.

The [Pushgateway](https://github.com/prometheus/pushgateway) is designed to take in metrics from a [service-level batch job](https://prometheus.io/docs/practices/pushing/) just before it exits, and serve those up to Prometheus until the next run of the batch job completes. If you try to do `sum_over_time(records_processed[1d])` to calculate the total number of records processed over the past day you'll find the figure is wildly inflated, as it'll count the records for every scrape of the Pushgateway rather than every push by the batch job.

The good news is that as of 0.4.0 the Pushgateway includes a metric called `push_time_seconds` which makes this possible when combined with recording rules:

groups:
  - name: example
    rules:
    - record: records\_processed\_dedupe
      expr: >
        # records\_processed if there was a push since the
        # last eval, otherwise return 0.
          (
             records\_processed
           and 
             last\_push\_time\_seconds{job="myjob"} != push\_time\_seconds{job="myjob"}
          )
        or
          push\_time\_seconds{job="myjob"} \* 0         
    - record: last\_push\_time\_seconds
      expr: max\_over\_time(push\_time\_seconds{job="myjob"}\[5m\])

Now `sum_over_time(records_processed_dedupe[1d])` will return how many records were processed over the past day. This approach isn't perfect as there could be a failed push, too frequent pushes, or Prometheus could be down at just the wrong time - however metrics only need to be good enough to make engineering decisions on rather than 100% accurate. If you need something that is 100% accurate, a logs-based monitoring system is a better choice.

To get a little into the nitty-gritty I use the `last_push_time_seconds` metric here rather than `idelta` to detect a change as it is more resilient to issues such as a missed rule evaluation. This relies on Prometheus 2.x rule groups evaluating rules in order. These expressions are constructed so that if the group is deleted from the Pushgateway, then `records_processed_dedupe` will no longer be set as `push_time_seconds` won't return anything. The `max_over_time`  makes this resilient to failed scrapes of the Pushgateway. This is one thing to watch out for with self-referential rules, you don't want ever growing cardinality due to churn. This method has the same limitations as rate when it comes to counters appearing out of the blue with an existing non-zero value as was discussed in [Existential issues with metrics](https://www.robustperception.io/existential-issues-with-metrics/), so will not count the very first push.

_Want help monitoring batch jobs? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
