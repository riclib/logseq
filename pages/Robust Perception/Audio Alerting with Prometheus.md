---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/audio-alerting-with-prometheus
author: [[Brian Brazil]] 
---
# Audio Alerting with Prometheus

> Prometheus offers integrations with systems like PagerDuty, Email and Hipchat for alert notifications - but what if you want do something that's not supported out of the box? The Alertmanager's generic web hook has got you covered.

---
Prometheus offers integrations with systems like PagerDuty, Email and Hipchat for alert notifications - but what if you want do something that's not supported out of the box? The Alertmanager's generic web hook has got you covered.

I'd like to be told when my internet connections go down, rather than finding out the hard way. To do this I setup alerts for connection failure, and had my media centre speak the text of alert notifications.

```html
<iframe loading="lazy" src="https://www.youtube.com/embed/f9gr7CHO_aY" width="560" height="315" frameborder="0" allowfullscreen="allowfullscreen"></iframe>
```
For the sake of brevity, I'm going to assume you already know how to download and compile Prometheus binaries. First off, I run the [blackbox_exporter](https://github.com/prometheus/blackbox_exporter) as root:
```shell

sudo ./blackbox_exporter
```

This will take in HTTP requests from Prometheus, do an ICMP ping and return the result.

Next I create a [Prometheus](http://prometheus.io/) configuration, and run it:

```yaml
cat <<'EOF' > prometheus.yml
global:
  scrape_interval: 1s
  evaluation_interval: 1s

rule_files:
  - internet.rules
scrape_configs:
  - job_name: 'blackbox_magnet'
    metrics_path: /probe
    params:
      module: [icmp]
      target: [8.8.4.4]
    static_configs:
      - targets:
        - 127.0.0.1:9115
    relabel_configs:
      - source_labels: []
        regex: (.*)
        target_label: instance
        replacement: Magnet
  - job_name: 'blackbox_upc'
    metrics_path: /probe
    params:
      module: [icmp]
      target: [8.8.8.8]
    static_configs:
      - targets:
        - 127.0.0.1:9115
    relabel_configs:
      - source_labels: []
        regex: (.*)
        target_label: instance
        replacement: UPC
alerting:
  alertmanagers:
  - static_configs:
    - targets:
      - "127.0.0.1:9093"
    
EOF
cat <<'EOF' > internet.rules
groups:
- name: internet.rules
  rules:
  - alert: InternetDown
    expr: probe_success == 0
    annotations:
      DESCRIPTION: '{{$labels.instance}} internet connection down'
      SUMMARY: '{{$labels.instance}} down'
EOF
./prometheus
```
Once per second Prometheus will poll the blackbox_exporter to see if the connections are working, and if not will fire alerts to the alertmanager. I've configured my firewall so that 8.8.4.4 and 8.8.8.8 go out via the Magnet and UPC internet connections respectively.

I need an alertmanager to take in alerts and send alert notifications:
```shell
cat <<'EOF' > alertmanager.conf
notification_config {
  name: "audio"
  webhook_config {
    url: "http://192.168.5.42:1234/"
    send_resolved: true
  }
}
aggregation_rule {
  repeat_rate_seconds: 3600
  notification_config_name: "audio"
}
EOF
./alertmanager
```
And finally on my media box, a small python script to receive and process the notifications:
```shell
cat <<'EOF' > alarm.py
#!/usr/bin/python

import cgi
import json
import pipes
import subprocess
from BaseHTTPServer import BaseHTTPRequestHandler
from BaseHTTPServer import HTTPServer

class AlarmHandler(BaseHTTPRequestHandler):
    def do_POST(self):
        self.send_response(200)
        self.end_headers()
        data = json.loads(self.rfile.read(int(self.headers['Content-Length'])))
        text = "%s %s" % (data["alert"][0]["summary"], data["status"])
        print text
        subprocess.call(
            "espeak %s -p 10 --stdout | aplay -Dhdmi_complete -" % pipes.quote(text), shell=True)

if __name__ == '__main__':
   httpd = HTTPServer(('', 1234), AlarmHandler)
   httpd.serve_forever()

EOF
python alarm.py
```
The script uses [eSpeak](https://en.wikipedia.org/wiki/ESpeak) to convert the alert summary to audio. There's no limit to what you can do with alert notifications using the generic web hook.

_Note: The alert rule and old alertmanager configuration formats have changed since this was first posted, so this will no longer work out of the box with the latest binary versions._
