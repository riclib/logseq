---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/setting-thresholds-on-alerts
author: [[Brian Brazil]] 
---
> Alert thresholds can be surprisingly tricky to get right.

# Setting Thresholds on Alerts


Alert thresholds can be surprisingly tricky to get right.

Let's say you're fully on board with [symptom based alerting](https://docs.google.com/document/d/199PqyG3UsyXlwieHaqbGiWVa8eMWi8zzAn0YfcApr8Q/edit) and you've a latency SLO/SLA of say 500ms over a month for some service which has a typical value of 200ms (whether that is the average or some percentile is not relevant here). What threshold should you set for your paging latency alert?

You could set 500ms over an hour, but that would likely be too slow to fire to be useful and also too slow to stop firing once whatever issue is resolved (which is irritating, though not the end of the world). You could set 300ms over a minute, but brief spikes would be spammy.

On one hand you want to spot and resolve issues before they threaten your SLO, or even worse persist for long enough that the SLO the entire month can no longer be met. On the other hand you don't want to interrupt an engineer unnecessarily in the middle of their sleep or work. That could lose you hours of productivity, or contribute to alert fatigue.

What I'd personally do is look at the graph of the value over the past few weeks, take the highest value seen over that time period outside of an outage, add a buffer, round up generously, and then set the alert threshold to that. I'd usually look over a 5 minute period, with a `for` of at least 5 minutes. This should produce an alert threshold that should only fire if something is very wrong with few false positives, and is thus worth interrupting a human for.

It's possible that a value produced by this method is above the SLO value or is too high to catch issues before they threaten the SLO. At that point you'd need to evaluate how many false positives are okay, and if maybe the SLO is unrealistic and needs to be relaxed.

There are other knobs you can tune too. False positives are more likely when you're servicing few requests, such as at night, as a small number requests from weird/automated/abusive traffic that are usually in the noise can have a disproportionate effect. [Adding a condition](https://www.robustperception.io/combining-alert-conditions) to the alert that the request rate is some minimum value across the service can help with that, without leaving you completely blind during the night if there's a sudden surge of user traffic. Bumping the `for` value can also help with reducing alert noise.

Other controls can also help, not every SLO issue has to be dealt with via a page to the oncall. Rather than having over sensitive alerts to catch any possible threat from slow-burn issues, you could regularly check what your daily/weekly/monthly SLO values are currently looking like. This might be a once a day task for the oncall engineer, or something quickly covered in your weekly operations meeting. This is also an opportunity for longer term trends to be spotted before they become an SLO issue.

In the end keep in mind that defining alerts and their thresholds is not a one off thing. You'll need to maintain and update them over time in response to changes in the system, traffic, and how acceptable the current alerting load on your engineers is.

_Have questions about reducing alert noise? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
