---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/how-to-unit-test-prometheus-instrumentation
author: [[Brian Brazil]] 
---
# How to unit test Prometheus instrumentation

> If you've determined a metric should be tested, how do you go about that?

---
If you've determined a metric should be tested, how do you go about that?

While you shouldn't automatically test metrics, Prometheus client libraries offer facilities to make it as easy as possible to do so.

Taking [Python](https://github.com/prometheus/client_python/) as an example, let's say we had a counter that we wanted to check was incremented:
```python
from prometheus_client import Counter
my_counter = Counter('my_counter_total', 'A useful help string.')

def my_function():
    my_counter.inc()
```

To test this we compare the before and after values:
```python
import unittest
from prometheus_client import REGISTRY

class TestMyFunction(unittest.TestCase):
    def test_metric_incremented(self):
       before =  REGISTRY.get_sample_value('my_counter_total')
       my_function()
       after = REGISTRY.get_sample_value('my_counter_total')
       self.assertEqual(1, after - before)
```
For gauges that are incremented/decremented and histograms you can use the same technique. For gauges that are set, you can simply compare the values.

The same approach works for [Java](https://github.com/prometheus/client_java) using the [getSampleValue](https://prometheus.io/client_java/io/prometheus/client/CollectorRegistry.html#getSampleValue-java.lang.String-) method of [CollectorRegistry.defaultRegistry](https://prometheus.io/client_java/io/prometheus/client/CollectorRegistry.html#defaultRegistry).

Prometheus metrics are usually file-level global variables, registered with a global registry. While you could plumb around the registry to use, this would cause friction and decrease the number of useful metrics that'd end up being added. The global nature of metrics does not make them harder to unit test. In fact it reduces the chances that you'll miss a bit of plumbing, resulting in the metric and tests passing perfectly but not being exposed.
