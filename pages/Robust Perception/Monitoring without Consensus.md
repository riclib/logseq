---
created: [[Aug 5th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/monitoring-without-consensus
author: [[Brian Brazil]] 
---
> When designing a monitoring system and the datastore that goes with it, it can be tempting to go straight for a clustered highly consistent approach. But is that the best approach?

# Monitoring without Consensus


When designing a monitoring system and the datastore that goes with it, it can be tempting to go straight for a clustered highly consistent approach. But is that the best approach?

Monitoring, like all other systems, is a question of engineering tradeoffs. For a medical system you'd probably choose to go for a completely reliable system with 100% accurate data, with a corresponding velocity and financial cost. Most of us don't have the luxury of being able to throw that much time and money at our systems, so we have to chose what we want to focus on.

For a monitoring system that focus is going to be firstly reliability, as you want to be able to continue using your monitoring system when the rest of your infrastructure is falling down around you. Secondly you need to be able to have data that's good enough to make operational decisions.

## Clustering is Hard

It's in this context of reliability that we look at clustering, such as CP clustering as exemplified by Paxos and Raft. These are [really, really, really hard to get right](https://aphyr.com/tags/jepsen) and it is often many years after a project is launched that all the main issues are handled. On one project I worked on it took over two years after it was feature complete to work out all the kinks related to clustering, and that was a relatively simple system.

CP clustering systems are designed to stop working when there's major network issues, as they prioritise consistency over availability. It tends to be events like network blips and partitions that trigger bugs - and [the network is not reliable](https://aphyr.com/posts/288-the-network-is-reliable). Bugs are also likely to lurk in the intricate code required to implement a clustered distributed system. So it seems like clustering may not be the best thing to have on the critical path of your monitoring system.

## Other Options

So if we're not going to cluster what can we do instead? Whatever about reliability, don't we need clustering to scale? Are we doomed to a lifetime of painful manual management of our monitoring service?

The good news is that the answers to these questions is no. Clustering is needed when the processing you want won't fit on one machine, and the good news with Prometheus is that it's so efficient that you can almost always fit the working set for a given service on one machine. At 800,000 samples per second, a single Prometheus can easily monitor over 10,000 machines. Beyond that you can shard by team, which you would do [anyway for organisational reasons](http://www.robustperception.io/scaling-and-federating-prometheus/); and as a bonus it gives you protection from isolation issues and cascading failures.

So what about reliability then? If there's no clustering how do I ensure my alerts keep working? The Prometheus solution is simple, run [two identical Prometheus servers](http://www.robustperception.io/prometheus-and-alertmanager-architecture/) that both go and scrape targets and send alerts.

The next question is usually, "But won't they have slightly different data"? And the answer is yes, they will have different data. My question to you is, does it matter?

## My God, It's Full of Races

The thing is that monitoring is chock full of race conditions. Maybe the network card had a slightly fuller buffer than last time, or the servers aren't quite NTP synced, or the traffic went down a different network path, or the kernel decided to preempt Prometheus just as it was about to do a scrape. On a larger scale there could be a network blip for a few minutes, or a server restart.

All of this is normal, and these are things your alerts and queries need to deal with. If you had hair-trigger alerts you'd quickly discover that they produce a lot of useless, spammy notifications due to these races. Similarly if your prediction engine isn't stable in the face of a small bit of noise in the data then the problem isn't with the noise.

## Perfection

The followup question is usually, "If one of the two identical servers has a blip, can I backfill the data from the other one to remove the gap?" There's two issues here. Firstly, what would be the right semantics to use here? Due to the fact that there's been a failure we can't know which server and time ranges in which to trust the data. Secondly, this would be a minor form of clustering, with all the risks that brings. As one example as we backfill in the data that'd likely double the load on both Prometheus servers, which could cause a far worse outage than the original blip.

Instead let's look at the question from a different standpoint. You are monitoring a system that has an availability target somewhere in the 2-4 9's range. In that context do you really need your monitoring system to be more than 4 9's reliable? A few minutes long blip once a quarter isn't the end of the world. Given that 99% of monitoring data isn't accessed after a day, you'll likely not care about it in a week's time.

Whitebox monitoring is not a perfect record of the behaviour of your system, and it doesn't need to be a perfect record. Don't get caught in the trap of trying to make it perfect. Instead embrace that races and outages happen, and build robust monitoring that behaves gracefully in the real world.

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
