---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/monitoring-kafka-in-a-docker-container-using-prometheus
author: [[Brian Brazil]] 
---
# Monitoring Kafka in a Docker container using Prometheus

> Want to learn how to monitor a Kafka instance that's inside of a Docker container?

---
Want to learn how to monitor a Kafka instance that's inside of a Docker container?

Previously we've looked at [monitoring Kafka using Prometheus](https://www.robustperception.io/monitoring-kafka-with-prometheus/). Like the last post on this topic, we'll be using the JMX exporter to expose Kafka's metrics for our Prometheus to scrape.

This time however, Kafka and the JMX exporter Java agent will be inside of a Docker container.

This blogpost assumes that you already have [Docker](http://docker.com/) and [Docker Compose](https://docs.docker.com/compose/) installed on your machine.

Begin by grabbing the [example code](https://github.com/RobustPerception/docker_examples/tree/master/prometheus_kafka) which contains a Docker setup that will spin up Zookeeper (a Kafka dependency), a Kafka instance, the JMX exporter agent, and a Prometheus instance to monitor it all.

Do this by running `docker-compose up` inside of the `prometheus_kafka/` directory.

You should now be able to head over to [Prometheus](http://127.0.0.1:9090/) and query some Kafka, JMX, and JVM metrics.

[![](https://www.robustperception.io/wp-content/uploads/2017/09/Screen-Shot-2017-09-04-at-17.31.47-640x253.png)](https://www.robustperception.io/wp-content/uploads/2017/09/Screen-Shot-2017-09-04-at-17.31.47.png)

If you wish to edit the Prometheus configuration file, you can find it under `mount/prometheus/prometheus.yml`. It's currently set to simply scrape the running JMX exporter agent.

The ports and host name of each component may be configured inside of `docker-compose.yml` for a non localhost setup.
