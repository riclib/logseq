---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/there-are-100000-seconds-in-a-day
author: [[Brian Brazil]] 
---
# There are 100,000 Seconds in a Day

> Just after you've launched is not the best time to find out that you can't handle the load you predicted, or that running costs are much higher than you'd like. By estimating the operational parameters of your system as you design you can gain confidence that the system will work as you expect.

---
Just after you've launched is not the best time to find out that you can't handle the load you predicted, or that running costs are much higher than you'd like. By estimating the operational parameters of your system as you design you can gain confidence that the system will work as you expect.

When building your product, there's often times you're worried that your solution won't scale -  so you'll add a cache. Or a database. Or sharding. Or a more expensive machine.

These all have a cost to implement. If in hindsight they weren't needed, or weren't enough and lead to an outage, then you've lost time and money that could have been spent more productively.

How to avoid this? Get into the habit of doing a quick estimate when making a design decision. The idea is to get a  rough notion in less than a minute, it doesn't have to be perfectly accurate. Round generously towards the worst case, there are 100,000 seconds in a day, a disk seek takes 10ms, a million read requests to S3 costs $1.

There are three possible outcomes from the estimate:

1.  The design is well within the required bounds
2.  The design is well outside of bounds
3.  It's not clear the design is within bounds, more precise analysis is required

If it's the first then all is good, and you can continue designing with confidence. If it's well outside of bounds then you haven't wasted time on the idea, and can move on to alternatives. Finally if it's unclear, only then do you need to spend a few minutes doing a more precise calculation.

Let's take an example. You want to have a hundred machines read data once a second. If the data is read S3 that's 100k seconds times 100 machines = 10M requests per day = $100 per day or $36k per year. That's probably too expensive. You could add a cache, but is there an easier way? With a seek time of 10ms you can handle 100 requests per second, so a single hard disk could handle this for far cheaper!

This technique lets you save time and money. As an added bonus, as this becomes an integral part of your design process you'll find yourself regularly choosing an appropriate solution first time around.
