---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/a-silence-is-a-snooze
author: [[Conor Broderick]] 
---
# A Silence is a Snooze

> Have you ever wondered what exactly that "Silence" button on each of your alerts in the Alertmanager actually does? Perhaps you have an idea but are unsure of their correct usage or value.
In this post I aim to clear up any confusion surrounding the silencing of alerts, so you can make the most of its functionality and understand when and why to use them.

---
Have you ever wondered what exactly that "Silence" button on each of your alerts in the Alertmanager actually does? Perhaps you have an idea but are unsure of their correct usage or value.  
In this post I aim to clear up any confusion surrounding the silencing of alerts, so you can make the most of its functionality and understand when and why to use them.

[![](https://www.robustperception.io/wp-content/uploads/2017/05/Alarm-clock-2-600x402.png)](https://www.robustperception.io/wp-content/uploads/2017/05/Alarm-clock-2.png)

As the title of this post states, a silence similar to that of the snooze feature of your alarm clock that wakes you up every morning to get out of bed for work.  
When you silence an alert, you're not scheduling the Alertmanager to not alert you of a particular event for certain periods of the day, or indefinitely ignoring the firing of a particular alert. It's simply a tool for the engineers responding to the alert to not have it ringing in their ear while they try to deal with the issue causing it in the first place.

Like your alarm in the morning, it only ever completely stops when you get up out of bed (if you're using it as intended!). Similarly, alerts fired by the Alertmanager only ever stop once their causes have been dealt with. Using silences allows us to simply mute alerts for a given amount of time while we investigate the problem causing them. By silencing we are acknowledging their existence and that we are now aware of them.

Another scenario where you may find silencing your alerts useful is in the case of maintenance work. In order to prevent the unnecessary firing of alerts for machines you know are going to be down for maintenance, one could silence these alerts for the time period they'll be worked on to avoid pointless alerts being sent out to your team. In this case silences are preventing unnecessary noise in your team's alerting notifications.