---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/the-three-types-of-cache
author: [[Brian Brazil]] 
---
# The Three Types of Cache

> Caches are a common feature of distributed systems, often added to improve performance. There are three main types of cache, and knowing about them will help you design robust systems.

---
Caches are a common feature of distributed systems, often added to improve performance. There are three main types of cache, and knowing about them will help you design robust systems.

A cache is a subset of your data that remembers values you're previously fetched or calculated from somewhere further away or more expensive. The next time around, if you're looking for the same piece of data again then you can get it from the cache which is called a "cache hit". If the data you want isn't in the cache that's called a "cache miss", and you have to fetch or calculate it from the source.

When adding a cache it's usually done for one of three reasons: latency, capacity or availability. Caches added for one purpose also have the behaviour of the others, so it's important to be aware of their characteristics and how cache misses can cause unexpected problems.

## Latency

Latency is one of the most common reasons a cache is added. Let's say the majority of your requests are for only a small subset of your data on disk. You could wait 10ms every time for a disk seek, but if you could put the frequently accessed data in RAM and be many orders of magnitude faster for those requests.

A cache will make your average latency lower. Cache misses still take the same amount of time, so make sure your timeouts allow for the time a cache miss takes. In case there's lots of misses, memory buffers and connection limits should allow for more in-progress requests.

## Capacity

Capacity is the other common reason for a cache being added. Continuing the above example, with 10ms per disk seek you can only service 100 requests per second. As most requests now only hit RAM, you can service many more requests per second with the same disk.

This advantage of a capacity cache is also its biggest downside. If the cache get flushed such as by a machine reboot, you don't get the capacity back until the cache has warmed back up. This can cause a problem called a [thundering herd](https://en.wikipedia.org/wiki/Thundering_herd_problem), where lots of requests (some of them for the same data) are all trying to use the disk beyond the disk's capacity to serve them.

Ways of handling this include ensuring only a portion of the cache is flushed at once, having other servers able to handle the additional load if one or two go down and lose their cache, ensuring identical requests only result in one request to the disk, or load shedding where you drop requests on the floor rather than getting completely overloaded.

## Availability

Availability caches are less common. If the disk above was at risk of being removed, having a cache would mean you can still serve some requests even though all cache misses fail. This is often a consideration with network services, as for example being unable to talk to a remote server because DNS had a brief outage would rarely be a good tradeoff.

Serving data without talking to the source can lead to using stale data. Approaches for dealing with this include deleting entries from the cache after a time, regular refreshes, asynchronously updating the cache at each request and having a separate process that remove or updates the caches when values change.

## Summary

Caches can be a net gain in your systems when the appropriate considerations are taken into account. A caches added for one purpose will have all three properties:

-   Latency: Reduces latency, cache misses still take the same time and resources
-   Capacity: Reduces resource needs, have to allow for cache suddenly being empty
-   Availability: Reduces downtime, data can be stale

In practice empty caches and stale data tend to be the most complex aspects to manage.
