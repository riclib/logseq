---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/unlimited-costs-limited-revenue
author: [[Brian Brazil]] 
---
# Unlimited costs, Limited revenue

> This week Microsoft removed unlimited storage from their OneDrive offering, because surprise surprise people were using it as unlimited storage. Does your product have features that cost you time and money, without your users paying accordingly?

---
This week Microsoft [removed unlimited storage from their OneDrive offering](https://blog.onedrive.com/onedrive_changes/), because surprise surprise people were using it as unlimited storage. Does your product have features that cost you time and money, without your users paying accordingly?

When starting out and trying to gain traction, it's normal to add lots of features to draw in more users. Maybe you're building a chat application, and you add the ability to send pictures. At a time when your servers are idle and costs are low, this seems perfectly innocent.

Small things like this can come back to bite you. You probably calculated your hosting costs based on a few kilobytes of text per day, not megabytes of images. It could turn out that this is negligible in the grand scheme of things, or it could turn out to be a major cost. If each user shares 10MB worth of images a day, that's 0.3 GB a month which is about one cent per user per month in S3. That compounds over time, as the next month there's new images plus all the previous months. The first year it'll cost $0.78 and the second year it's $3.00. If your business model is a fixed monthly fee per user that could be a big problem!

What's more problematic is abuse. Abuse is anything where anyone can hurt you, your customers or the public by using your product in a way that wasn't intended. In this case abuse may take the form of using your chat application as as a generic file transfer and storage mechanism. It's unlikely after all that you'll have implemented any garbage collection to delete old files, especially when building a minimum viable product.

It pays to look out for anywhere where your pricing model doesn't align with your costs, but that doesn't mean that you should be paralysed by it. A more balanced approach is to be aware of the various ways your costs could go off-kilter, and check once a month to see if any of them are starting to rear their head. At that point you can instigate counter-measures for real problems, rather than fretting over all the potential ways things could go wrong.
