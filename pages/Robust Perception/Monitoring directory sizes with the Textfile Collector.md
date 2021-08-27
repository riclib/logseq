---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/monitoring-directory-sizes-with-the-textfile-collector
author: [[Brian Brazil]] 
---
# Monitoring directory sizes with the Textfile Collector

> The node exporter includes many metrics out of the box, it can't possibly cover all use cases though. That's where the textfile collector comes in, allowing you to extend machine instrumentation for your use case.

---
The [node exporter](https://github.com/prometheus/node_exporter) includes many metrics out of the box, it can't possibly cover all use cases though. That's where the textfile collector comes in, allowing you to extend machine instrumentation for your use case.

The node exporter by default exposes metrics about filesystem fullness. Sometimes though there's particular directories you want to keep an eye on such as `/var/log`. There's too many directories to sanely store them all to [Prometheus](https://prometheus.io/), but we can expose a few important ones.

Previously I showed how to use the textfile collector for [sensor metrics](http://www.robustperception.io/quick-sensor-metrics-with-the-textfile-collector/). A similar technique can be used to track directory sizes. Add the following to `/etc/cron.d/directory_size`:
```
*/5 * * * * root du -sb /var/log /var/cache/apt /var/lib/prometheus | sed -ne 's/^([0-9]+)t(.*)$/node_directory_size_bytes{directory="2"} 1/p' > /var/lib/node_exporter/textfile_collector/directory_size.prom.$$ && mv /var/lib/node_exporter/textfile_collector/directory_size.prom.$$ /var/lib/node_exporter/textfile_collector/directory_size.prom
```
How does this work?

The first bit says to run this command every 5 minutes as the root user. `du` reports the disk usage of its arguments. `-s` means summarize, it won't display the usage of subdirecotries. `-b` returns the value in bytes. The `sed` command formats the output of `du` in the  way the textfile collector expects,making the directory a label. We don't have to worry about escaping as none of our directories have unusual characters. This is redirected to a temporary file, and then atomically renamed. This presumes that the `--collector.textfile.directory` flag of the node exporter is set to `/var/lib/node_exporter/textfile_collector`.

When Prometheus scrapes the node exporter, the node exporter reads this file and returns the samples. This can then be graphed from Prometheus:

[![](http://www.robustperception.io/wp-content/uploads/2015/11/Screenshot-231115-174543.png)](http://www.robustperception.io/wp-content/uploads/2015/11/Screenshot-231115-174543.png)

Disk usage by Prometheus data directory

One advantage of this technique is not just that you can expose more metrics from the node exporter, but that you can do so without having to run the node exporter as root so that it can read all the required directories. It's always better not to have your network services running as root!
