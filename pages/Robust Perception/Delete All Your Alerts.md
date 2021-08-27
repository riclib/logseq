---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/delete-all-your-alerts
author: [[Brian Brazil]] 
---
> Trying to improve alerting piecemeal can be difficult.

# Delete All Your Alerts


Trying to improve alerting piecemeal can be difficult.

If you're in a situation where you are being inundated with low-value alerts, trying to gradually improve things can be a never-ending struggle. I once joined a team that was getting a few hundred email alerts per week, which the oncall was meant to handle all of. This was unsurprisingly leading to some burnout, but even worse every now and then a big outage was being missed as it was lost in the noise - if there were even alerts about it. That lead to more alerts being created to in order try and catch that exact outage if it ever happened again, and often undoing any recent gains in reducing alert noise.

The problem with tackling alerts individually is that each alert on its own usually has a good reason to exist, some corner case it catches even if it's a little noisy. In aggregate this can lead to a very poor signal to noise ratio. While it's true that if that alert had existed prior to the previous outage it might have let you know in time to mitigate the outage, right though they're now spamming the oncall about something that is not user impacting.

The way I handled this was a total reset, the oncall would no longer have to care about any of those alerts. Instead I configured some basic symptom-based error rate, latency, and blackbox alerts to go to the oncall via a more appropriate paging mechanism. The results? The number of alerts the oncall had to care about dropped from hundreds a week down to maybe one or two. There was no discernible change to the level of outages (things were not yet at a stage where there were SLIs to measure this more formally). So overall a clear win.

Sometimes when dealing with a suboptimal alerting situation, the best approach is to figure out what alerts you need from first principles.

_Looking to reduce alerting noise? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
