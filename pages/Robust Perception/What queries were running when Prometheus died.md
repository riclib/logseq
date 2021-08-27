---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/what-queries-were-running-when-prometheus-died
author: [[Brian Brazil]] 
---
> As of Prometheus 2.12.0 there's a new feature to help find problematic queries.

# What queries were running when Prometheus died?


As of Prometheus [2.12.0](https://www.robustperception.io/new-features-in-prometheus-2-12-0) there's a new feature to help find problematic queries.

While Prometheus has [many features](https://www.robustperception.io/limiting-promql-resource-usage) to limit the potential impacts of expensive PromQL queries on your monitoring, it's still possible that you'll run into something not covered or there aren't sufficient resources provisioned.

As of Prometheus 2.12.0 any queries which were running when Prometheus shuts down will be printed on the next startup. As all running queries are cancelled on a clean shutdown, in practice this means that they'll be printed only if Prometheus is OOM killed or similar. On the next startup you'll see log lines like these:

level=info ts=2019-08-28T14:30:09.142Z caller=main.go:329 msg="Starting Prometheus" version="(version=2.12.0, branch=HEAD, revision=43acd0e2e93f9f70c49b2267efa0124f1e759e86)"
level=info ts=2019-08-28T14:30:09.142Z caller=main.go:330 build\_context="(go=go1.12.8, user=root@7a9dbdbe0cc7, date=20190818-13:53:16)"
level=info ts=2019-08-28T14:30:09.142Z caller=main.go:331 host\_details="(Linux 4.15.0-55-generic #60-Ubuntu SMP Tue Jul 2 18:22:20 UTC 2019 x86\_64 mari (none))"
level=info ts=2019-08-28T14:30:09.142Z caller=main.go:332 fd\_limits="(soft=1000000, hard=1000000)"
level=info ts=2019-08-28T14:30:09.142Z caller=main.go:333 vm\_limits="(soft=unlimited, hard=unlimited)"
level=info ts=2019-08-28T14:30:09.143Z caller=query\_logger.go:74 component=activeQueryTracker msg="These queries didn't finish in prometheus' last run:" queries="\[{\\"query\\":\\"changes(changes(prometheus\_http\_request\_duration\_seconds\_bucket\[1h:1s\])\[1h:1s\])\\",\\"timestamp\_sec\\":1567002604}\]"
level=info ts=2019-08-28T14:30:09.144Z caller=main.go:654 msg="Starting TSDB ..."level=info

Here there was only one query running when Prometheus died unnaturally (which I had to go out of my way to make slow so it'd show up), so it's a likely culprit if Prometheus ran out of resources. However there could have been other queries running that had only trivial usage, or indeed it could have been that something else entirely triggered the termination, so a query being in this list doesn't automatically mean it's a problem.

If the issue was an overly expensive query and you can't just throw resources at it, some flags you may wish to tweak are `--query.max-concurrency`,  `--query.max-samples`, and `--query.timeout`.

_Wondering how to optimise your PromQL? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
