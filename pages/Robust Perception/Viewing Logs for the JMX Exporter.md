---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/viewing-logs-for-the-jmx-exporter
author: [[Brian Brazil]] 
---
# Viewing Logs for the JMX Exporter

> A blog on monitoring, scale and operational Sanity

---
A blog on monitoring, scale and operational Sanity

Sometimes mBeans produce errors when scraped by the [JMX exporter](https://github.com/prometheus/jmx_exporter). Being able to look at detailed logs can help you figure out exactly which mBean is having issues and why.

Create a file called `logging.properties` with this content:

```
handlers=java.util.logging.ConsoleHandler
java.util.logging.ConsoleHandler.level=ALL
io.prometheus.jmx.level=ALL
io.prometheus.jmx.shaded.io.prometheus.jmx.level=ALL
```
Add the following flag to your Java invocation:

```
-Djava.util.logging.config.file=/path/to/logging.properties
```
If you run your application you'll now see logs on standard error.

If you already have a log handler such as logback, log4j or slf4j you should adjust their configs along the lines shown.
