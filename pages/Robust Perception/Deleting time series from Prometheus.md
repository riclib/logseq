---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/deleting-time-series-from-prometheus
author: [[Brian Brazil]] 
---
> If a misconfiguration leads to unwanted time series, it'd good to know how to remove them.

#  Deleting time series from Prometheus
If a misconfiguration leads to unwanted time series, it'd good to know how to remove them.

Whether it's a mistake in your relabelling rules or an incorrectly exposed metric, sometimes you want to remove data from Prometheus and don't want to wait until it hits the retention period. In this post we'll look at the [delete\_series API](https://prometheus.io/docs/prometheus/latest/querying/api/#delete-series) Prometheus 2.x has for this.

To use it ==you need to enable the admin api== by passing the `--web.enable-admin-api` command line flag to Prometheus, then you can provide selectors for the time series you wish to remove:
```shell
curl -X POST -g 'http://localhost:9090/api/v1/admin/tsdb/delete\_series?match\[\]=a\_bad\_metric&match\[\]={region="mistake"}'
```
The `-g` disables [curl's globbing functionality](https://ec.haxx.se/cmdline-globbing.html). Like the federation endpoint, any series which matches any selector will be affected. You can also pass `start`/`end` parameters to limit the time period affected, as by default it works over all time.

Once the request returns, the data will no longer be accessible. ==Storage space on disk will be freed up at the next compaction==. You can hurry this along by using the clean tombstones API, but this is not something you should be doing on a regular basis.