---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/be-discerning-in-what-dashboards-you-share-with-users
author: [[Brian Brazil]] 
---
> There's no way that sharing metrics with your users or customers can go wrong. Right?

# Be discerning in what dashboards you share with users


There's no way that sharing metrics with your users or customers can go wrong. Right?

Having a status page so that users can check your services are working without having to bother your support staff during an outage is a good practice, which is getting more common. Sharing too much can cause problems too though.

I've seen it happen more than once that a user of a system would overly-fixate on one particular graph or metric, possibly related to some previous real issue, and all future problems both real and imagined will be tied to it. This can cause your support staff to have to respond to issues that don't exist, and delay resolution of real problems.

As an example while oncall I once had a developer contact me pointing at a graph indicating that a metric was high for a certain subsystem in one datacenter. It did have a spike, however that datacenter was not serving traffic at that time, the metric was still well within SLO, and all other datacenters were normal - so this wasn't the problem.

In this way, a little knowledge is can be a dangerous thing. If you don't understand a system well enough to appreciate the subtleties and relevance of a given metric, it's easy to come to incorrect conclusions. Humans are really good at spotting patterns after all, [including ones that don't exist](https://en.wikipedia.org/wiki/Apophenia).

When sharing dashboards beyond a core group try to focus on SLO/SLAs related items that tie to their experience, and avoid unrelated metrics that might help you but would confuse others. Keep in mind also that [what gets measured is what gets managed](https://en.wikipedia.org/wiki/Goodhart%27s_law), which can lead to perverse incentives and worse outcomes for everyone!

_Looking for advice on what's important to measure? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
