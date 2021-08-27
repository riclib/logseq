---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/running-into-burning-buildings-because-the-fire-alarm-stopped
author: [[Brian Brazil]] 
---
# Running into burning buildings because the fire alarm stopped

> At what point should you consider an alert resolved?

---
At what point should you consider an alert resolved?

If you've got an alerting system spamming you with low-value alerts that aren't actual problems, ignoring alerts that are no longer firing is a defence mechanism to deal with the operational overload you're experiencing. This however is pretty far from the ideal of how alerts should be handled.

Every alert and notification that gets to you saps some of your attention, distracting you from being a good engineer and working on long-term project work. So make every alert count!

That you're willing to ignore an alert at all is in indication that the alert should probably not have fired in the first place. So you should never think that handling of an alert is complete merely because it is no longer firing. You should still dig into it to see if there's an underlying problem. While alerts do sometimes fire spuriously due to unusual circumstances, if an alert does so 2-3 times in a row then it's definitely time to relax the threshold on the alert - or maybe even consider deleting the alert, or rejigging the alerting design.

In accordance with [My Philosophy on Alerting](https://docs.google.com/document/d/199PqyG3UsyXlwieHaqbGiWVa8eMWi8zzAn0YfcApr8Q/edit), alerts should require intelligent human action. In addition, pages should require you to react with urgency. If it ever gets to the stage where you receive a page and go "Oh, it's just that subsystem again", and don't feel the need to immediately deal with it, then things have gone wrong.

In this context you should understand that the Prometheus Alertmanager does not deal with human responses to alerts, only whether the alert is still firing in Prometheus. Tracking whether an alert is still under investigation, part of a larger issue, or has already been fixed is a matter for an incident management system. Tools such as PagerDuty, OpsGenie, VictorOps or a ticketing system are good choices for this. It would be unwise to close off an issue in your incident management system when the alert stops firing in Prometheus, so avoid using resolved notifications from the Alertmanager in this manner.

Just as you'd wait for the all clear before returning to a building where the fire alarm went off, investigate the cause of your alerts before deeming them resolved.
