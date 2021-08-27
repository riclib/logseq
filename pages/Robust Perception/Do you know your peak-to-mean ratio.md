---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/do-you-know-your-peak-to-mean-ratio
author: [[Brian Brazil]] 
---
# Do you know your peak-to-mean ratio?

> Traffic from users to your servers isn't a steady stream, it waxes and wanes over the day and week. The peak-to-mean ratio is your primary tool to avoid outages or unnecessary costs due to this.

---
Traffic from users to your servers isn't a steady stream, it waxes and wanes over the day and week. The peak-to-mean ratio is your primary tool to avoid outages or unnecessary costs due to this.

Users aren't constantly glued to their screens, they have to go to work, go home and go to sleep. Users for most applications are also spread across multiple time-zones. The result of all this is peaks and troughs in your traffic, which can easily lead to a 3-4X difference between minimum and maximum load over the day.

Failing to account for this can lead to under-provisioning and outages. For example if you're expecting 10 million requests per day, that's an average of about 115 requests per second. If you only build out capacity for 115 requests per second, you're going to have a bad time at the next peak.

This can also lead to over-provisioning, which wastes resources. If you've a peak of 115 requests per second each of which needs 1MB to be stored, 10TB would be too much as you don't have to store as many requests in the troughs.

The peak-to-mean ratio is the key number you need to know, as it lets you safely convert from peak traffic to average traffic. To calculate yours, simply divide your peak traffic per second by your average traffic per second over a day or week. For a globally distributed user base somewhere in the range of 1.6 to 1.7 is typical.

Let's look at the above examples again. `10M / 86400 * 1.7` is almost 200 requests per second - a big difference in how many servers you'll need to build out. Similarly `115 / 1.7 * 86400 * 1MB` is about 6TB, a significant storage saving.

Know that you know how, always apply the peak-to-mean ratio when going from per second/minute peak traffic to and from per day/week/month traffic. This will give you the correct result, saving money and reducing outages!

_(Aside: While there's [there are 100,000 seconds in a day](http://www.robustperception.io/there-are-100000-seconds-in-a-day/) when doing rough numbers at the design stage, it pays to be a little more precise for provisioning and capacity planning)._
