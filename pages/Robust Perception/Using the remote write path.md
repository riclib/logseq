---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/using-the-remote-write-path
author: [[Brian Brazil]] 
---
# Using the remote write path

> Recent versions of Prometheus added an experimental remote write feature. Let's take a look.

---
Recent versions of Prometheus added an experimental remote write feature. Let's take a look.

Prometheus's local storage isn't intended as a long term data store, rather as more of an ephemeral cache. This is as storing effectively unbounded amounts of time series data would require a distributed storage system, whose [reliability characteristics](https://www.robustperception.io/monitoring-without-consensus/) would not be what you want from a monitoring system. Instead the intention is that a separate system would handle durable storage and allow for seamless querying from Prometheus.

The [remote write](https://prometheus.io/docs/operating/configuration/#%3Cremote_write%3E) path is one half of thus. It allows for each sample that's ingested from scrapes, and calculated from rules, to be sent out in real time to another system. This could be long term storage, or an adaptor that sends to something like Kafka for further processing.

So let's give it a spin. This is currently all experimental, so we're using the simplest thing which will work - which is to say Protocol Buffers over HTTP. This is very likely to change in the future.

I'm going to assume you already have a working Go environment:

```shell
go get -d -u github.com/prometheus/prometheus/{cmd/prometheus,documentation/examples/remote_storage/example_write_adapter}
cd ${GOPATH}/src/github.com/prometheus/prometheus/documentation/examples/remote_storage/example_write_adapter
go run server.go &
```
This will run the [demo write adapter](https://github.com/prometheus/prometheus/blob/master/documentation/examples/remote_storage/example_write_adapter/server.go) which prints out the samples it is sent.

Now we just need to run a Prometheus that is pointing at it:

```shell
cd ${GOPATH}/github.com/prometheus/prometheus
make
cat <<EOF > prometheus.yml
global:
  scrape_interval: 5s
remote_write:
  - url: "http://localhost:1234/receive" scrape_configs:
  - job_name: prometheus
    static_configs:
    - targets: ['localhost:9090']
EOF
./prometheus
```

After a few seconds you'll see the samples in the output of the adapter.

This is just a simple example, but you're free to expand it in whatever way you like. You can also use relabelling to restrict what time series are sent out.

It is planned that the existing experimental Graphite/OpenTSDB/InfluxDB write support will be removed in favour of this new generic interface.
