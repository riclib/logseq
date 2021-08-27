---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/reloading-prometheus-configuration
author: [[Brian Brazil]] 
---
# Reloading Prometheus’ Configuration

> A common question from new users is if they need to restart Prometheus every time they change the configuration. The good news is that you don't, allowing your monitoring to continue uninterrupted as your system changes.

---
A common question from new users is if they need to restart [Prometheus](https://prometheus.io/) every time they change the configuration. The good news is that you don't, allowing your monitoring to continue uninterrupted as your system changes.

There are two ways to ask Prometheus to reload it's configuration, a `SIGHUP` and the POSTing to the `/-/reload` handler.

To send a SIGHUP, first determine the process id of Prometheus. This may be in a file such as `/var/run/prometheus.pid`, or you can use tools such as pgrep to find it. Then use the kill command to send the signal:
```shell
kill -HUP 1234
```
Alternatively, you can send a HTTP POST to the Prometheus web server:
```shell
curl -X POST http://localhost:9090/-/reload
```
Note that as of Prometheus 2.0, the `--web.enable-lifecycle` command line flag must be passed for HTTP reloading to work.

If the reload is successful Prometheus will log that it has updated its targets:
```
INFO[0248] Loading configuration file prometheus.yml source=main.go:196
INFO[0248] Stopping target manager... source=targetmanager.go:203
INFO[0248] Target manager stopped. source=targetmanager.go:216
INFO[0248] Starting target manager... source=targetmanager.go:114
```
If there's a syntax error the logs will something look like this:
```
INFO[0296] Loading configuration file prometheus.yml source=main.go:196
ERRO[0296] Couldn't load configuration (-config.file=prometheus.yml): yaml: line 1: mapping values are not allowed in this context source=main.go:208
```
You can use the `promtool` command that comes with Prometheus to syntax check your configuration and rules.
