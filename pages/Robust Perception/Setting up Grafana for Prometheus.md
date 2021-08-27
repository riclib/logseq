---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/setting-up-grafana-for-prometheus
author: [[Brian Brazil]] 
---
# Setting up Grafana for Prometheus

> A blog on monitoring, scale and operational Sanity

---
A blog on monitoring, scale and operational Sanity

[Grafana](http://grafana.org/) recently added support for [Prometheus](https://prometheus.io/). Let's take a look at how to get it up and running.

[![Grafana Prometheus Dashboard](http://www.robustperception.io/wp-content/uploads/2015/11/Screenshot-011115-132758-640x415.png)](http://www.robustperception.io/wp-content/uploads/2015/11/Screenshot-011115-132758.png)

First let's setup a quick Prometheus server to scrape itself so we have some metrics to play with:
```shell
wget https://github.com/prometheus/prometheus/releases/download/v2.0.0/prometheus-2.0.0.linux-amd64.tar.gz
tar -xzf prometheus-2.0.0.linux-amd64.tar.gz
cd prometheus-*
cat <<'EOF' > prometheus.yml
global:
 scrape_interval: 10s
 evaluation_interval: 10s
scrape_configs:
 - job_name: 'prometheus'
   static_configs:
    - targets:
      - localhost:9090
EOF
./prometheus &
```
Next let's get Grafana up and running:
```shell
wget https://grafanarel.s3.amazonaws.com/builds/grafana-latest.linux-x64.tar.gz
tar -xzf grafana-latest.linux-x64.tar.gz
cd grafana-*
./bin/grafana-server &
```

## Adding Prometheus Data Source

1.  Go to [http://localhost:3000](http://localhost:3000/)
2.  Enter the username `admin` and password `admin`, and then click "Log In".
3.  Click "Data Sources" on the left menu
4.  Click "Add new" on the top menu
5.  Add a default data source of type `Prometheus` with `http://localhost:9090` as the URL

[![](http://www.robustperception.io/wp-content/uploads/2015/11/Screenshot-011115-125025-640x330.png)](http://www.robustperception.io/wp-content/uploads/2015/11/Screenshot-011115-125025.png)

## Adding a Dashboard

1.  Click "Dashboards" on the left menu
2.  Click "Home" on the top menu, and then "+ New" at the bottom of the panel that appears
3.  Click the "Graph" box that appears underneath "New Dashboard" in the top left hand side of the screen to add a new graph panel
4.  Click on "Panel Title" and then "Edit" in the small box that appears above
5.  Enter `go_goroutines{job='prometheus'}` in the "Query" field
6.  Click the eye symbol on the right hand side of screen to see your graph
7.  Click the floppy disk icon on the top menu to save your dashboard

[![Adding a Graph of Goroutines](http://www.robustperception.io/wp-content/uploads/2015/11/Screenshot-011115-131437-640x406.png)](http://www.robustperception.io/wp-content/uploads/2015/11/Screenshot-011115-131437.png)

Adding a Dashboard of Goroutines

You can use any Prometheus expression in a query, plot multiple expressions and much more. Now that you're up and running go explore more of what's possible with Prometheus and Grafana!
