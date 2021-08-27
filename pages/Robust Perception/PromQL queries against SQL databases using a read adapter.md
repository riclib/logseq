---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/promql-queries-against-sql-databases-using-a-read-adapter
author: [[Brian Brazil]] 
---
# PromQL queries against SQL databases using a read adapter

> Prometheus 1.6 includes a new experimental feature called remote read. Let's look at what it can do.

---
Prometheus 1.6 includes a new experimental feature called remote read. Let's look at what it can do.

Long term storage is one of the most requested features of Prometheus. The [remote write path](https://www.robustperception.io/using-the-remote-write-path/) allows streaming data out of Prometheus, and the new remote read allows pulling that data back in PromQL queries.

While long term storage is its primary intended use, the APIs don't restrict it to that. You could use it to transparently access data from an older monitoring system that Prometheus has replaced, or even a SQL database. Such uses should be very limited on both reliability and sanity grounds, but let's run small example to give a taste of what's possible.

Presuming you already have a working Go build environment, let's run the adapter:
```
go get -d github.com/robustperception/go_examples/sql_read_adapter
cd $GOPATH/src/github.com/robustperception/go_examples/sql_read_adapter
go build
./sql_read_adapter &
```
This creates and fills a local [SQLite](https://www.sqlite.org/) database.

Now let's point a minimal Prometheus at it which only does remote read:
```shell
wget https://github.com/prometheus/prometheus/releases/download/v2.0.0/prometheus-2.0.0.linux-amd64.tar.gz
tar -xzf prometheus-*.tar.gz
cd prometheus-*
cat <<'EOF' > prometheus.yml
remote_read:
 - url: http://localhost:8080/read
   read_recent: true
EOF
./prometheus
```
If you go to Prometheus's expression browser, you can run arbitrary SQL queries:

[![](https://www.robustperception.io/wp-content/uploads/2017/04/Screenshot_2017-04-06_14-04-35.png)](https://www.robustperception.io/wp-content/uploads/2017/04/Screenshot_2017-04-06_14-04-35.png)

The [source code](https://github.com/RobustPerception/go_examples/blob/master/sql_read_adapter/main.go) for this demo is available, and is easily extendable to other databases and data sources. Don't go too far though, you don't want a misbehaving system to take out your critical monitoring!
