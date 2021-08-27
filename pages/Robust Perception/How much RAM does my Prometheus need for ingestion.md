---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/how-much-ram-does-my-prometheus-need-for-ingestion
author: [[Brian Brazil]] 
---
# How much RAM does my Prometheus need for ingestion?

> It can be a little confusing to figure out Prometheus memory usage. Let's break part of it down.

---
It can be a little confusing to figure out [Prometheus](https://prometheus.io/) memory usage. Let's break part of it down.

I've been doing loadtests to better understand how Prometheus behaves in both big and small deployments. From this I've been able to distil some simple rules to help guide you in sizing your Prometheus for ingestion. [Due](https://github.com/prometheus/prometheus/pull/2292) [to](https://github.com/prometheus/prometheus/pull/2295) [several](https://github.com/prometheus/prometheus/pull/2297) [improvements](https://github.com/prometheus/prometheus/pull/2322) [I](https://github.com/prometheus/prometheus/pull/2333) [made](https://github.com/prometheus/prometheus/pull/2321) [as](https://github.com/prometheus/common/pull/70) a consequence of these loadtests, these results apply only to Prometheus 1.5.x. Prometheus 1.6.x has a different set of flags, but the general principles still apply.

There's two numbers you need to determine. The first is how many chunks you need as a persistence buffer, that is data that's ready and waiting to be written to disk. Writing out data immediately to potentially millions of files wouldn't scale, so Prometheus batches them in memory and writes them out in a cycle that takes 6 hours or so. To calculate this, wait until your Prometheus has been running for at least 6 hours and then evaluate  `(increase(prometheus_local_storage_chunk_ops_total{job="prometheus",type="create"}[6h]) / 2 / .8 * 1.6)`. This gives the value of your `-storage.local.max-chunks-to-persist` flag. We halve to allow for the cycling going through, and then divide by .8 to avoid rushed mode and then give 60% slack to allow for variation over time.

The second number is how many unique time series you expect to be ingesting in any given 6 hour period. `max_over_time(prometheus_local_storage_memory_series{job="prometheus"}[6h])` is a good estimate of this.

Add the two numbers together, and that's the minimum for your `-storage.local.memory-chunks` flag. This fits chunks that are still being filled, plus the ones that are already filled and waiting to be written.

You can expect RSS RAM usage to be at least 2.6kiB per local memory chunk. The chunks themselves are 1024 bytes, there is 30% of overhead within Prometheus, and then 100% on top of that to allow for Go's GC.

## Complications

The above is generally correct for a typical setup. There's quite a few caveats though.

The first one is a big one, this only covers ingestion. Running queries will require additional RAM, both for any additional chunks pulled in from disk and for evaluating the expression.

Next up, the 6h cycle time isn't quite the case. It's actually 10% of your retention period, limited to 6h. So if you've a retention period under 60h, adjust the above numbers accordingly.

More relevant to most is that the 6h cycle time is only a target, it'll be slower than that by the actual time it takes to do the persistence itself. The slack in the .6 should mostly cover this. I'd additionally suggest setting `-storage.local.checkpoint-dirty-series-limit` to the same as `-storage.local.memory-chunks` to avoid additional checkpointing, though this increases how long crash recovery may take.

This all assumes that the persistence cycle is keeping up, which you may have difficulty doing on a non-SSD or if you've lots of checkpointing. You could choose to make it faster by decreasing the maximum number of chunks to persist, at the cost of additional iops. This will generally cause rushed mode to kick in, which disables fsyncs and doesn't wait between series like it usually does. If you wish to disable fsync during normal persistence you can do so with `-storage.local.series-sync-strategy=never`.

