---
created: [[Aug 4th, 2021]]
type: #clipping
status: #reviewed
tags: prometheus 
source: https://www.robustperception.io/shutting-down-prometheus
author: [[Brian Brazil]] 
---
> Shutting down Prometheus properly helps reduce the risk of delays at startup. So how do you do that?

# Shutting down Prometheus
Shutting down Prometheus properly helps reduce the risk of delays at startup. So how do you do that?

==If not shutdown cleanly Prometheus should in theory be able to recover gracefully at startup, however it may take longer or you might hit an obscure bug somewhere in the software stack which causes you problems==. ==Thus it's best to ask Prometheus to shutdown, and then wait however long it takes to stop - which is usually not much time at all==.

Prometheus is a standard Unix binary, so ==can be asked to shutdown using a `SIGTERM`==. You will need to know the process id of Prometheus, which you might be able to find with `pgrep -f prometheus` or in a file such as `/var/run/prometheus.pid` depending on your setup. Then you can run:
```shell
kill -TERM 1234
```
where 1234 is the pid.

==You can also shutdown Prometheus via HTTP, which is the only option on Windows which does not support Unix signals such as `SIGTERM`==. For security reasons you need to pass the `--web.enable-lifecycle` flag to Prometheus for this to be enabled, and then you can do:
```shell
curl -X POST http://localhost:9090/-/quit
```
All going well it will return

Requesting termination... Goodbye!

and Prometheus itself will log something like
```logfmt
level=warn ts=2018-05-10T14:57:47.515721541Z caller=main.go:378 msg="Received termination request via web service, exiting gracefully..."
level=info ts=2018-05-10T14:57:47.515793883Z caller=main.go:398 msg="Stopping scrape discovery manager..."
level=info ts=2018-05-10T14:57:47.515832602Z caller=main.go:394 msg="Scrape discovery manager stopped"
level=info ts=2018-05-10T14:57:47.515819324Z caller=main.go:411 msg="Stopping notify discovery manager..."
level=info ts=2018-05-10T14:57:47.515931633Z caller=main.go:432 msg="Stopping scrape manager..."
level=info ts=2018-05-10T14:57:47.515965465Z caller=main.go:407 msg="Notify discovery manager stopped"
level=info ts=2018-05-10T14:57:47.516084719Z caller=main.go:426 msg="Scrape manager stopped"
level=info ts=2018-05-10T14:57:48.58685747Z caller=head.go:357 component=tsdb msg="WAL truncation completed" duration=1.212400909s
level=info ts=2018-05-10T14:57:48.59514324Z caller=manager.go:460 component="rule manager" msg="Stopping rule manager..."
level=info ts=2018-05-10T14:57:48.595355991Z caller=manager.go:466 component="rule manager" msg="Rule manager stopped"
level=info ts=2018-05-10T14:57:48.595393882Z caller=notifier.go:512 component=notifier msg="Stopping notification manager..."
level=info ts=2018-05-10T14:57:48.595580989Z caller=main.go:573 msg="Notifier manager stopped"
level=info ts=2018-05-10T14:57:48.59561159Z caller=main.go:584 msg="See you next time!"
```
before exiting.