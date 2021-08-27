---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/configuring-prometheus-with-docker
author: [[Brian Brazil]] 
---
# Configuring Prometheus with Docker

> It's easy to get a demo Prometheus up and running with Docker. How do you take it beyond that and provide your own configuration?

---
It's easy to get a demo [Prometheus](https://prometheus.io/) up and running with [Docker](https://docker.com/). How do you take it beyond that and provide your own configuration?

`docker run -p 9090:9090 prom/prometheus` is all is takes to get Prometheus running on [http://localhost:9090/](http://localhost:9090/). For production use though you'll want to specify your own configuration, and it's always good to use [source control](http://www.robustperception.io/do-you-have-basic-infrastructure/).

[![](http://www.robustperception.io/wp-content/uploads/2015/11/Screenshot-221115-000430.png)](http://www.robustperception.io/wp-content/uploads/2015/11/Screenshot-221115-000430.png)

Prometheus Console

Docker makes this easy with layers, allowing you to take an existing image and make changes on top of it. Let's start by creating a simple Prometheus configuration file:
```shell
cat <<EOF > prometheus.yml
global:
  scrape_interval:     10s
  evaluation_interval: 10s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

EOF
```
This tells the Prometheus server to scrape itself on port 9090.

Now let's create a Dockerfile that adds this on top of the `prom/prometheus` image:
```shell
cat <<EOF > Dockerfile
FROM prom/prometheus

# Add in the configuration file from the local directory.
ADD prometheus.yml /etc/prometheus/prometheus.yml
EOF
shell

Next we can build and run it:

docker build -t prometheus_simple .
docker run -p 9090:9090 prometheus_simple

If you visit [http://localhost:9090/consoles/prometheus.html](http://localhost:9090/consoles/prometheus.html) you'll see the Prometheus console!

You can add more files for rules and consoles, and check it all into source control. For example, the configuration for this post can be found on [Github](https://github.com/RobustPerception/docker_examples/tree/master/prometheus_simple).
