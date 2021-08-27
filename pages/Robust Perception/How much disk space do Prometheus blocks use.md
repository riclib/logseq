---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/how-much-disk-space-do-prometheus-blocks-use
author: [[Brian Brazil]] 
---
> Memory for ingestion is just one part of the resources Prometheus uses, let's look at disk blocks.

# How much disk space do Prometheus blocks use?


[Memory](https://www.robustperception.io/how-much-ram-does-prometheus-2-x-need-for-cardinality-and-ingestion) for ingestion is just one part of the resources Prometheus uses, let's look at disk blocks.

Every 2 hours Prometheus compacts the data that has been buffered up in memory onto blocks on disk. This will include the chunks, indexes, tombstones, and various metadata. The main part of this should usually be the chunks themselves, and you can see how much space each sample takes on average by graphing:

  rate(prometheus\_tsdb\_compaction\_chunk\_size\_bytes\_sum\[1h\])
/ 
  rate(prometheus\_tsdb\_compaction\_chunk\_samples\_sum\[1h\])

Combined with `rate(prometheus_tsdb_head_samples_appended_total[1h])` for the samples ingested per second, you should have a good idea of how much disk space you need given your retention window.

It's a bit more complicated though, as there's also indexes to consider. If you have lots of churn in your metrics these can end up taking a non-trivial amount of space. We can use the [jq](https://stedolan.github.io/jq/) tool on the `meta.json` in each block in Prometheus's data directory to get a better idea of the ratio between samples ingested and ultimate storage costs:

$ for i in \*/meta.json; do jq '.stats.numBytes / .stats.numSamples' $i; done
1.7751466111566343
1.7667422794818908

There will be additional compaction of the 2h blocks into bigger blocks, up to a few weeks (or 10% of your retention, whichever is smaller). This can result in smaller indexes, but the chunks will be the same size - less any samples removed due to tombstones. One effect of this is that if you're using time-based retention that if a block contains samples that are still inside the time range, then the entire block will be kept around until it is all outside of the retention window.

Given the per-block ratios between bytes and samples, your sample ingestion rate, your retention period, an extra 10% to allow for blocks that are straddling the retention period, plus another 10% for temporary space during compaction you can calculate your block storage needs with:

bytes per sample \* ingestion rate \* retention time \* 1.2

If you're using size-based retention, then you can reverse the above formula to estimate how much retention time you'll have.

This does not include the space taken by the WAL, which is a topic for another day.

_Have questions about scaling Prometheus? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
