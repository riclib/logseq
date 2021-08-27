---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/you-look-good-have-you-lost-machines
author: [[Brian Brazil]] 
---
# You look good, have you lost machines?

> Whether you're on bare metal or using a cloud provider, there's a question you should always be able to answer. What machines do I have, and what is meant to be running on them?

---
Whether you're on bare metal or using a cloud provider, there's a question you should always be able to answer. What machines do I have, and what is meant to be running on them?

Production infrastructure is not static. Services get created and obsoleted. Machines get brought up and turned-down. As organizations grow and systems get ever more complicated it's not too surprising then that sometimes machines get "lost", costing you money that's not being tracked anywhere. Conversely a service might have been running short a machine without you noticing.

The key to avoiding these problems is a well maintained machine and service database. From it you should be able to tell what machines you are meant to have, what services are on them and how many of each there are. This is one part of your [basic infrastructure](http://www.robustperception.io/do-you-have-basic-infrastructure/). On a regular basis you should reconcile what's in production with what the database says, and rectify any irregularities.

![](http://www.robustperception.io/wp-content/uploads/2015/12/You-look-good-have-you-lost-machines-.png)

Two services, one missing machine and one lost machine.

Modern service discovery mechanisms such as Consul can be used to find your machines and services, but aren't sufficient for the role of machine and service database. Consider a machine on which the Consul agent has died, or the machine has gone dodgy and is no longer accessible over the network. You won't be able to find that machine via Consul, however Consul would be useful as part of the reconciliation process.

Ah ha, you say, I'm on EC2 which provides me a list of all my running machines including the stopped and broken ones! That's true, what it doesn't tell you is that there's three auto-scaling groups that were meant to be deleted weeks ago. Or that some of your application servers are meant to have EBS, not instance store.

Whatever compute resources you have, make sure that they're recorded in a machine and service database. This helps you to:

-   Recreate the system in the event of failure
-   Spot incorrect configuration
-   Turn off resources that are surplus to requirements
