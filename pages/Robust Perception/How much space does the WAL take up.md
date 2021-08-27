---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/how-much-space-does-the-wal-take-up
author: [[Brian Brazil]] 
---
> The quoted storage numbers for Prometheus are usually for the blocks, not including the WAL.

# How much space does the WAL take up?


The quoted storage numbers for Prometheus are usually for the blocks, not including the WAL.

All samples that are ingested by Prometheus are written to the write ahead log or WAL, so that on restart in-memory state which hasn't made it to a block yet can be reconstructed. While this tends not to be massive compared against the blocks, it's not nothing either. Here we'll we looking at Prometheus 2.17.1.

The [WAL format is documented](https://github.com/prometheus/prometheus/blob/05442b31c899af5640f534c9e5d790ea58f18365/tsdb/docs/format/wal.md#record-encoding), and is composed of various types of record. Each sample record represents one scrape or one recording rule output. Similarly the series records are all the new time series from one scrape or recording rule, and will even be created if the sample append ends up being rolled back due to e.g. hitting the scrape sample limit. There's also tombstone records, however users deleting recent series should be a very rare occurrence so we can safely ignore them.

As the number of samples from one scrape tends to be relatively large in this context, we will ignore the 23 bytes of overhead for simplicity. Each sample has a variable length int for the diff from a base value for its id and timestamp. As all the series in a scrape will likely have been first seen at around the same time, with maybe some series created later on, so 4 bytes worth of varint seems a safe number to allow for churn (plus some wiggle room for overheads). All timestamps will typically be the same, so 1 byte is sufficient there. Finally the value itself is always 8 bytes. So that's around 13 bytes per sample ingested.

What about series? They're about 9 bytes overhead (a series is unlikely to have more than 127 labels and thus require a 2nd byte for the length) per series, plus the size of all the label names and values, plus two bytes overhead for each label (once again, a label name/value is unlikely to have more than 127 bytes). Doing some hand waving, that's going to be roughly about the same size as the full labelset rendered as a string.

Let's use a concrete example numbers. Say there was a Prometheus ingesting 100k samples/s that was also significantly churning with 1M series per 2 hours with 100 bytes for series labels. So it's 100000\*13 = 1.3MB per second for the samples, and 1000000/3600/2\*100 = 13.8kB per second for the series - which is to say that the series are not significant even with heavy churning.

So we know that we can simplify down to about 13 bytes of WAL per sample, how long are samples kept around? We don't want to use too much disk space, nor do we want to have to replay the entire WAL since the Prometheus first started to ensure we know about all relevant series. This brings us to checkpoints, which summarise older WAL entries down to just the series creations that may still be relevant. The checkpointing logic runs every two hours after head compaction, and will checkpoint the [oldest third](https://github.com/prometheus/prometheus/blob/05442b31c899af5640f534c9e5d790ea58f18365/tsdb/head.go#L680-L689) of segments. So summing the geometric series with a constant of 2 hours and a ratio of 2/3, we get about 4 hours worth of segments that will be kept around. Add then the 2 hours of WAL that'll build up until the next checkpoint, for 6 hours in total.

The size of the checkpoint must also considered. Let's say that the above 100k samples/s Prometheus had 1M old series that were in the checkpoint at 100 bytes each, that'd be 100MB - which is about the size of a segment so not notable.

So overall the size of the WAL will be around 6 hours worth of samples ingested, with 13 bytes used per sample. For the 100k samples/s Prometheus that's around 26GB of data, or around 10% of the size the blocks take for the default 2 week retention period.

As of Prometheus 2.11.0 there's also WAL compression, which you can enable with the `--storage.tsdb.wal-compression` flag. This should roughly half the size of the WAL, at the cost of some additional CPU. This is enabled by default as of 2.20.0.

Update: As of [Prometheus 2.18.0](https://www.robustperception.io/new-features-in-prometheus-2-18-0), 3 rather than 6 hours of WAL is kept around.
