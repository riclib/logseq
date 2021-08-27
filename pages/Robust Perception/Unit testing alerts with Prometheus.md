---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/unit-testing-alerts-with-prometheus
author: [[Brian Brazil]] 
---
> In the previous post we looked at testing rules. You can also test alerts.

# Unit testing alerts with Prometheus


In the [previous post](https://www.robustperception.io/unit-testing-rules-with-prometheus) we looked at testing rules. You can also test alerts.

Let's take a simple example of an alert with some templating:

wget https://github.com/prometheus/prometheus/releases/download/v2.5.0/prometheus-2.5.0.linux-amd64.tar.gz
tar -xzf prometheus-\*.tar.gz
cd prometheus-\*
cat >rules.yml <<'EOF'
groups:
 - name: example
   rules:
    - alert: MyAlert
      expr: avg without(instance)(up) < 0.75
      for: 2m
      labels:
        severity: page
      annotations:
        description: 'Only {{$value}} of {{$labels.job}} job is up'
EOF

To test this as before we create a `test.yml`, with the following content:

rule\_files:
  - rules.yml
evaluation\_interval: 1m
tests:
 - interval: 1m
   input\_series:
    - series: 'up{job="node",instance="foo"}'
      values: '1+0x10'
    - series: 'up{job="node",instance="bar"}'
      values: '1+0x5 0+0x5'
    - series: 'up{job="prometheus",instance="foo"}'
      values: '1+0x10'
   alert\_rule\_test:
    - alertname: MyAlert
      eval\_time: 7m
    - alertname: MyAlert
      eval\_time: 8m
      exp\_alerts:
       - exp\_labels:
           severity: page
           job: node
         exp\_annotations:
           description: 'Only 0.5 of node job is up'

Once again you can then run these tests with:

./promtool test rules test.yml

which should return:

Unit Testing: test.yml
  SUCCESS

This is the same input data as in the previous post, but here we're testing alerts instead. The first tests that no alerts are firing for `MyAlert` at 7 minutes in:

   alert\_rule\_test:
    - alertname: MyAlert
      eval\_time: 7m

The second tests that a single alert is firing a 8 minutes in, and that it has all the right labels and annotations:

    - alertname: MyAlert
      eval\_time: 8m
      exp\_alerts:
       - exp\_labels:
           severity: page
           job: node
         exp\_annotations:
           description: 'Only 0.5 of node job is up'

If you expected more alerts to be firing with the same alertname, you would list all of them under `exp_alerts`.

And that's all there is to it. You can test rules and alerts together, and there are more advanced features like specifying what order rule groups should be evaluated in to help simulate rules across federation hierarchies.

_Wondering what are good alerts to have? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
