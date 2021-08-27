---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/target-labels-are-for-life-not-just-for-christmas
author: [[Brian Brazil]] 
---
# Target labels are for life, not just for Christmas

> How should you choose the labels to put on your Prometheus monitoring targets? Let's take a look.

---
How should you choose the labels to put on your [Prometheus](https://prometheus.io/) monitoring targets? Let's take a look.

Target labels are a key aspect of scraping, as they tell you what the target is. They come from service discovery, which is combined with relabelling to extract the labels that make sense in your organisation.

There are a few common misconceptions and misunderstandings around how they are best used that I'd like to go over.

## Target Labels Should be Constant

As the target labels are the identity of the target, if you change any of them then that identity changes - and with it the time series you scrape from it. This would result in all your graphs, alerts and other expressions breaking.

So don't do that.

This means that you need your target labels to be the same from the time the thing being monitored is created until it no longer exists. For a machine that'd usually be from the day it's first turned on until the day it's thrown out. For a target running on a scheduling system such as Kubernetes that's the lifetime of the pod. For a part of a service running on a particular machine that's from when it's turned up to when it no longer runs on that machine.

But you may ask, how do I group targets by things like software version, machine owner, and other things that vary over the lifetime of a target? The answer is to include a time series in the scrape with this information, which we've looked at in [other](http://www.robustperception.io/how-to-have-labels-for-machine-roles/) [posts](http://www.robustperception.io/exposing-the-software-version-to-prometheus/).

This is not to say that target labels never change. It's not unusual that every few years you'll change your label taxonomy and all the targets need to be updated accordingly. For example you could go from pure bare metal to also using machines in the cloud, and now need to represent new concepts like availability zones. This is okay as it's a one-off breakage which is usually a good trade-off when balanced against the long-term benefits you get from the new naming scheme. This is not the case for targets which continuously change labels.

## Target Labels should be Minimal

Those used to event logging might feel the urge to put every possible available piece of data and metadata about the target as label on the target, so that they can do arbitrary dimensional analysis. Resist that urge. You don't want the CPU type, amount of RAM, application, OS, the name of the person who built the machine, and that it was built on a Tuesday as target labels, even if this is constant information.

Why not? Every label is one more label you need to remember when writing expressions that are used in graphs and alerts. Accidentally miss one in your `without` clause and boom, your alerting just broke. Every label on your time series makes it a little bit harder to work with and write expressions for, so make every target label count.

For the same reason the set of label names used on targets should be bounded and well-known. It shouldn't be possible for extra labels to pop out of nowhere without the forewarning and consent of the person running the monitoring, who may not be the same person running the target. It's not possible to deal with labels you don't know the names of, because you need to know their names to work with them and remove them.

The way to approach this is to ensure that your target labels are normalised. That is to say as you go from least to most specific, each label should distinguish this target from other targets in a way that can't be done without that label.

You'll usually have a datacenter, region, location or cluster label of some form. Even if you're only in one location right now you'll want to distinguish them when you grow in the future. A owner, team or service is common to indicate what the service is and who owns it, and as are env or environment to indicate production versus development.

You'll have some label to tell you the type of thing you're monitoring and where it is in the stack, typically just the `job` label fulfils this role and should be specified in almost every expression to avoid accidentally working on unrelated targets, but depending on the application you might have additional labels such as `shard`.

The `instance` label generally uniquely identifies a target, so there's no need to add additional `host` or `alias` labels on top of that to provide a prettier name - instead take advantage of the existing `instance` label and give it the better name.

If you want labels beyond this, you should look at taking the [machine role](http://www.robustperception.io/how-to-have-labels-for-machine-roles/) approach so that the labels are usable in expressions, without polluting the target labels.

Incidentally different teams/owners, clusters/regions and environments should probably have their own Prometheus server. While the labels mightn't be useful inside the Prometheus itself (and may even be `external_labels` rather than target labels), they're very useful for routing and silencing alerts.

## Conclusion

Target labels are a first-order way in which you can take advantage of the multi-dimensionality of the Prometheus data model. To make them as useful as possible and to avoid problems down the line, keep target labels constant across the lifetime of the target, and keep the number of target labels as low as you practically can.
