---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/monitoring-not-just-for-outages
author: [[Brian Brazil]] 
---
# Monitoring: Not Just For Outages

> It's common to think of monitoring as something just to alert you when things are going wrong.  At Robust Perception we believe in Inclusive Monitoring, where all aspects of systems are monitored and available to provide insight and drive decisions.

---
It's common to think of monitoring as something just to alert you when things are going wrong.  At Robust Perception we believe in Inclusive Monitoring, where all aspects of systems are monitored and available to provide insight and drive decisions.

[![](http://www.robustperception.io/wp-content/uploads/2015/10/Prometheus-Overview.png)](http://www.robustperception.io/wp-content/uploads/2015/10/Prometheus-Overview.png)

Monitoring isn't just about alerting at the edges

Monitoring means different things to different people; from manually looking at log files, to alerting on checks or even a room full of staff watching dashboards 24/7. No matter what it means to you, the most crucial aspect is likely getting timely alerts about things going awry.

But your monitoring can be much more than that.

Your monitoring shouldn't just tell you that there is a problem, but also help you determine why there is a problem. You don't want just to have an alert about your service being slow - you want to know if there was a traffic increase in the past hour, if machines have failed, if backend services are being slow or internal processing is taking longer.

[![](http://www.robustperception.io/wp-content/uploads/2015/10/Prometheus-Overview-1.png)](http://www.robustperception.io/wp-content/uploads/2015/10/Prometheus-Overview-1.png)

Inclusive Monitoring is about inside your applications

Beyond alerting and debugging, monitoring has a place as a core part of your engineering culture. Inclusive monitoring is about instrumenting all the logic in your code, not just the edges. Track how many widgets each request submits to the API, so that when you're designing a new storage system you know how much data you'll have to handle. Track hit rates of in-memory caches, so that you can size them up and down as traffic patterns change - or even break it out to Memcached to save RAM and increase hit rates. Track how often users hit the slow-path of your business logic, so you know when it's time to make it faster and when a new feature uses the slow-path that you're prepared for the latency hit.

These insights about how hundreds of little things in your code let you move faster, make better technical and business decisions, and ultimately increase your efficiency. This is what inclusive monitoring is about. Don't just monitor at the edges.
