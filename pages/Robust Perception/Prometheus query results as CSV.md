---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/prometheus-query-results-as-csv
author: [[Brian Brazil]] 
---
# Prometheus query results as CSV

> The default JSON output isn't always what you want when querying Prometheus. Let's see how to get out CSV files.

---
The default JSON output isn't always what you want when querying Prometheus. Let's see how to get out CSV files.

I've written a [small Python program](https://github.com/RobustPerception/python_examples/blob/master/csv/query_csv.py) to show how you can view the output of an [instant query](https://prometheus.io/docs/querying/api/#instant-queries) as CSV files.

Let's see it in action against the [live demo](http://demo.robustperception.io/):
```shell
pip install requests  # If you don't have it already.
wget https://raw.githubusercontent.com/RobustPerception/python_examples/master/csv/query_csv.py
python query_csv.py http://demo.robustperception.io:9090 'irate(process_cpu_seconds_total[1m])'
```
Which produces output like:
```
name,timestamp,value,instance,job
,1477858669.096,0.0030060120239314473,demo.robustperception.io:9093,alertmanager
,1477858669.096,0.510999999998603,demo.robustperception.io:9090,prometheus
,1477858669.096,0.0020082337584533144,demo.robustperception.io:9100,node
,1477858669.096,0.0009992006395102197,demo.robustperception.io:9091,pushgateway
```
This can be further processed or redirected to a file, which you can load up in Excel!
