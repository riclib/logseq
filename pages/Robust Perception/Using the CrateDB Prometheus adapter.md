---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/using-the-cratedb-prometheus-adapter
author: [[Brian Brazil]] 
---
# Using the CrateDB Prometheus adapter

> While Prometheus' local storage provides us with a simple storage setup it is limited to the amount of data that can fit on one machine.

---
While Prometheus' local storage provides us with a simple storage setup it is limited to the amount of data that can fit on one machine.

Recently several long term storage options for Prometheus have come on the scene. In this blogpost we'll look at setting up the [CrateDB Prometheus Adapter](https://github.com/crate/crate_adapter), which we developed on behalf of CrateDB. It will accept Prometheus remote read/write requests, and send them to be stored in [CrateDB](https://crate.io/download/).

Begin by downloading and running [CrateDB:](https://crate.io/download/#testing) (note CrateDB v2.2 is required!)

```shell
wget https://cdn.crate.io/downloads/releases/crate-2.2.0.tar.gz
tar -xzf crate-2.2.0.tar.gz
cd crate-2.2.0/bin
./crate
```
This will download, install and start the CrateDB service on your local machine at port 4200.

Next we'll create the following table in our CrateDB:

(You can paste snippet below into the CrateDB console over at [localhost:4200/#/console](http://localhost:4200/#/console))
```sql
CREATE TABLE "metrics" (
  "timestamp" TIMESTAMP,
  "labels_hash" STRING,
  "labels" OBJECT(DYNAMIC),
  "value" DOUBLE,
  "valueRaw" LONG,
  PRIMARY KEY ("timestamp", "labels_hash")
);
```
Next download and build the CrateDB Prometheus Adapter:

go get github.com/crate/crate_adapter
cd ${GOPATH-$HOME/go}/src/github.com/crate/crate_adapter
go build
./crate_adapter

The adapter will listen on port 9268 bey default, and talk to the local CrateDB running on port 4200. This is configurable via command line flags, which you can see by passing the -h flag.

Finally we'll download and run Prometheus, configuring its remote read and write paths to talk to the adapter, and for it to scrape itself so we'll have some metrics to play with:

```shell

wget https://github.com/prometheus/prometheus/releases/download/v2.0.0/prometheus-2.0.0.linux-amd64.tar.gz
tar -xzf prometheus-*.linux-amd64.tar.gz
cd prometheus-*
cat > prometheus.yml << EOF
global:
   scrape_interval: 5s
scrape_configs:
  - job_name: prometheus
    static_configs:
      - targets: ["localhost:9090"]
remote_write:
   - url: http://localhost:9268/write
remote_read:
   - url: http://localhost:9268/read
EOF
./prometheus
```

Prometheus should now start writing samples to CrateDB!

Head over to [http://localhost:4200/#/tables/doc/metrics](http://localhost:4200/#/tables/doc/metrics) and you should see something like the following in the top left hand corner of the screen which indicates that Prometheus is writing successfully to CrateDB:

[![](https://www.robustperception.io/wp-content/uploads/2017/09/Screen-Shot-2017-09-25-at-14.54.08.png)](https://www.robustperception.io/wp-content/uploads/2017/09/Screen-Shot-2017-09-25-at-14.54.08.png)

We can also verify this by heading over to the [CrateDB console](http://localhost:4200/#/console) and querying for our metrics:

[![](https://www.robustperception.io/wp-content/uploads/2017/09/Screen-Shot-2017-09-25-at-14.55.56-600x450.png)](https://www.robustperception.io/wp-content/uploads/2017/09/Screen-Shot-2017-09-25-at-14.55.56.png)

If we were to stop Prometheus, wipe its storage and start it back up again we'd still have seamless access to our data!
