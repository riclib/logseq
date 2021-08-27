---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/configuring-prometheus-storage-retention
author: [[Brian Brazil]] 
---
> How can you control how much history Prometheus keeps?

# Configuring Prometheus storage retention


How can you control how much history Prometheus keeps?

Prometheus stores time series and their samples on disk. Given that disk space is a finite resource, you want some limit on how much of it Prometheus will use. Historically this was done with the `--storage.tsdb.retention` flag, which specifies the time range which Prometheus will keep available. This is a minimum, so it'll keep an entire block if some of it is still within the retention window. If you know your ingestion rate in samples per second then you can multiply it by the typical bytes per sample (1.5ish, 2 to be safe) and the retention time to get an idea of how much disk space will be used.

As of Prometheus [2.7](https://www.robustperception.io/new-features-in-prometheus-2-7-0) and 2.8 there's new flags and options. 2.7 introduced an option for size-based retention with `--storage.tsdb.retention.size`, that is specifying a maximum amount of disk space used by blocks. This is still a bit best effort though as it does not (yet) include the space taken by the WAL or blocks being populated by compaction. To be safe allow for space for the WAL and one maximum size block (which is the smaller of 10% of the retention time and a month).

How do all these flags interact though? The behaviour was made more obvious in 2.8. There are three flags: `--storage.tsdb.retention.size`, `--storage.tsdb.retention.time`, and `--storage.tsdb.retention`. `--storage.tsdb.retention` is deprecated, superseded by `--storage.tsdb.retention.time` for clarity. This table summarises how they work together:

`--storage.tsdb.  
retention.size`

`--storage.tsdb.  
retention.time`

`--storage.tsdb.  
retention`

Result

Not set

Not set

Not set

Default 15d retention applies.

Not set

20d

Not se

20d retention applies.

Not set

Not set

10d

10d retention applies.

Not set

20d

10d

20d retention applies.

1TB

Not set

Not set

1TB size retention applies, no time limit.

1TB

20d

Not set

1TB size and 20d time retention apply - which ever happens first.

1TB

20d

10d

1TB size and 20d time retention apply - which ever happens first.

As you can see the `--storage.tsdb.retention.time` overrides the deprecated `--storage.tsdb.retention`, and both time and size based retention can be in force at once.

One common confusion I'd also like to cover is that Prometheus storage retention is not limited to 15 days. That's merely a default that was chosen many years ago, two storage backends back. As the above shows you can have it higher than that, or even not apply at all.

_Need help with Prometheus storage? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
