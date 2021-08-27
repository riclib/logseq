---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/why-do-resolved-notifications-contain-old-values
author: [[Brian Brazil]] 
---
> It often confuses users as to why resolved notifications don't contain updated annotations values. Let's dig into why.

# Why do resolved notifications contain old values?


It often confuses users as to why resolved notifications don't contain updated annotations values. Let's dig into why.

Let's say you had an alert like:

\- alert: MyAlert
  expr: foo > 100
  annotations:
    description: 'Foo is {{$value}}'

As the alert is firing, the Alertmanager will receive annotations such as "Foo is 102" and "Foo is 153", and use these in notifications. However an resolved notification from the Alertmanager can't contain an annotation like "Foo is 97".

The reason is that alerts in Prometheus aren't simple thresholds applied to values, they can be [far more complex](https://www.robustperception.io/combining-alert-conditions). To allow you this freedom, any sample which results from evaluating the alert expression becomes an alert and the sample including its labels and values are available for use in annotation templating. So if there's multiple `foo` metrics above 100, you'll get an alert for each of them, each of which will individually have their annotation templates expanded.

When a `foo` metric drops below 100, that sample will no longer be returned from the alerting expression and the alert is no longer firing. As there's no sample, there's no annotation templates expanded. Prometheus will detect that an alert has gone away, and inform the Alertmanager that the alert with the given labels is resolved.

The Alertmanager knows that the alert is resolved, but neither Prometheus nor the Alertmanager know what the current value of the series that caused the alert to fire is - if there even was exactly one series that was the cause. So when sending resolved notifications, the Alertmanager will use the annotations from the last firing alert it saw.

This is why resolved notifications contain old values, and is one of several [things to be wary of with resolved notifications](https://www.robustperception.io/running-into-burning-buildings-because-the-fire-alarm-stopped). To see the current value, a better way is to link to a dashboard from your notifications.

_Want advice on alerting best practices? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
