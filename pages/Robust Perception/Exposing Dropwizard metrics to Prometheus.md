---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/exposing-dropwizard-metrics-to-prometheus
author: [[Brian Brazil]] 
---
# Exposing Dropwizard metrics to Prometheus

> If you've an existing instrumentation library in use, it's not always practical to immediately switch to a Prometheus instrumentation library. There are a multitude of integrations available to aid your transition.

---
If you've an existing instrumentation library in use, it's not always practical to immediately switch to a Prometheus instrumentation library. There are a multitude of integrations available to aid your transition.

[Dropwizard metrics](http://metrics.dropwizard.io/) is a popular instrumentation library for the JVM. While it lacks labels and tends to do calculations in the client rather than exposing the raw counters that Prometheus prefers, you may wish to use what you already have while you're still busy porting instrumentation over to the Prometheus [Java client](https://github.com/prometheus/client_java).

The good news is that integrating the two instrumentation systems is only a few of lines of code. I'll presume you already have both the Prometheus and Dropwizard client libraries setup and working inside your application.

Add the `simpleclient_dropwizard` module to your `pom.xml`, or equivalent:

<dependency>
  <groupId>io.prometheus</groupId>
  <artifactId>simpleclient_dropwizard</artifactId>
  <version>0.0.23</version>
</dependency>

And then the following code:

import io.prometheus.client.CollectorRegistry;
import io.prometheus.client.dropwizard.DropwizardExports;

// Inside an initialisation function.
CollectorRegistry.defaultRegistry.register(new DropwizardExports(metrics));

Where `metrics` is a Dropwizard `MetricsRegistry`. And that's it!

A complete working example can be found on [Github](https://github.com/RobustPerception/java_examples/blob/master/java_dropwizard/src/main/java/io/robustperception/java_examples/JavaDropwizard.java).

_Need help transitioning metrics to Prometheus? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
