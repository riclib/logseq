---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/when-to-alert-with-prometheus
author: [[Brian Brazil]] 
---
> Alerting is an art. One must be sure to alert just enough to be aware of all problems arising in the monitored system while at the same time not drown out the signal with excess noise. In this blogpost we'll explain some of the best practices to use when alerting with Prometheus.

# When to Alert with Prometheus


Alerting is an art. One must be sure to alert just enough to be aware of all problems arising in the monitored system while at the same time not drown out the signal with excess noise. In this blogpost we'll explain some of the best practices to use when alerting with Prometheus.

Let's start with a simple question: "when to alert?" Alert when something important is broken and needs to be fixed ASAP, and alert when something important may break in the near future requiring human inspection. But what makes something important? An important system/service is one that will result in poor end-user experience. Alerting primarily on the potential pain points for your end-users will keep the number of alerts small preventing over-monitoring of the system as well as an increased importance of the alert received by the human on the other end. Keeping the number of alerts small and important obeys a key principle on alerting; namely that alerts should be important, actionable, and real. There's no point waking up the on-call team in the middle of the night for alerts they'll ignore.

Breaking this down further, let's explore what we should be alerting on for different types of systems.

For online services delivering content to our end users we would recommend alerting on conditions like [high latency](https://www.robustperception.io/avoid-outages-beware-the-knee/) and error rates as high up in the stack as possible. For capacity we'd suggest alerts to detect when [you will run out of space](https://www.robustperception.io/reduce-noise-from-disk-space-alerts/) that will lead to an outage. For batch jobs, alert when a job has not succeeded recently enough. Alert if the job has not complete in the time it would take for the job to complete twice if you can withstand more than a single failure.

So now that we know what to generally alert on, what should our alerts consist of? We recommend that alerts should link to relevant dashboards and consoles that answer the basic questions about the service being alerted on so the on-call engineer can quickly decipher what the underlying issue is.

In order to prevent some unnecessary noise in your alerting, allow for some extra slack time for alerting conditions to prevent alerting on small blips in the system.

Finally it's important to monitor your monitoring! One cannot be completely certain of the monitoring of their system unless the monitoring system itself is up and running.

_Interested in getting the best insights with Prometheus? [Contact us](mailto:prometheus@robustperception.io)._

Published by Conor Broderick in [Posts](https://www.robustperception.io/category/posts)
