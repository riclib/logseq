---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/unit-testing-rules-with-prometheus
author: [[Brian Brazil]] 
---
> As of 2.5.0, promtool has a feature to allow you to test your recording rules.

# Unit testing rules with Prometheus


As of [2.5.0](https://github.com/prometheus/prometheus/releases/tag/v2.5.0), promtool has a feature to allow you to test your recording rules.

PromQL is a powerful language, and it's possible to write expressions that [don't quite do what you expect](https://twitter.com/_codesome/status/1050780173350580224). As with programming generally, unit tests can help you detect such problems and reduce the chances of inadvertent breakage in the future. The unit test feature of promtool is based on the unit tests used by the PromQL implementation internally.

Let's take a simple example:

wget https://github.com/prometheus/prometheus/releases/download/v2.5.0/prometheus-2.5.0.linux-amd64.tar.gz
tar -xzf prometheus-\*.tar.gz
cd prometheus-\*
cat >rules.yml <<EOF
groups:
 - name: example
   rules:
    - record: job:up:sum
      expr: sum without(instance)(up)
EOF

To test this rule you can create `test.yml` with the following content:

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
   promql\_expr\_test:
    - expr: job:up:sum
      eval\_time: 1m
      exp\_samples:
       - labels: 'job:up:sum{job="node"}'
         value: 2
       - labels: 'job:up:sum{job="prometheus"}'
         value: 1
    - expr: job:up:sum
      eval\_time: 6m
      exp\_samples:
       - labels: 'job:up:sum{job="node"}'
         value: 1
       - labels: 'job:up:sum{job="prometheus"}'
         value: 1

You can then run these tests with:

./promtool test rules test.yml

which should return:

Unit Testing: test.yml
  SUCCESS

Let's break down the test file:

rule\_files:
  - rules.yml
evaluation\_interval: 1m

This says that we want to load the rules file `rules.yml` and evaluate the rules in it every minute.

tests:
 - interval: 1m
   input\_series:
    - series: 'up{job="node",instance="foo"}'
      values: '1+0x10'
    - series: 'up{job="node",instance="bar"}'
      values: '1+0x5 0+0x5'
    - series: 'up{job="prometheus",instance="foo"}'
      values: '1+0x10'

This defines the first set of tests, and provides input data. The series will have samples every minute. The first series is `1 1 1 1 1 1 1 1 1 1`, the second `1 1 1 1 1 0 0 0 0 0`, and the third is `1 1 1 1 1 1 1 1 1 1`.

   promql\_expr\_test:
    - expr: job:up:sum
      eval\_time: 1m
      exp\_samples:
       - labels: 'job:up:sum{job="node"}'
         value: 2
       - labels: 'job:up:sum{job="prometheus"}'
         value: 1
    - expr: job:up:sum
      eval\_time: 6m
      exp\_samples:
       - labels: 'job:up:sum{job="node"}'
         value: 1
       - labels: 'job:up:sum{job="prometheus"}'
         value: 1

Finally there's the actual tests. The first evaluates `job:up:sum` at one minute, which returns 2 and 1. The second evaulates it at six minutes, by which time `up{job="node",instance="bar"}` has become 0 and the result is now 1 and 1.

While this is a trivial example, it shows how you can go about testing more complex PromQL expressions.

_Have a question about PromQL? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
