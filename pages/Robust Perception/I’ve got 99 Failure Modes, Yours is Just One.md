---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/ive-got-99-failure-modes-yours-is-just-one
author: [[Brian Brazil]] 
---
# I’ve got 99 Failure Modes, Yours is Just One

> When running a production system there's an endless stream of issues that have the potential to cause you significant hassle. How should you deal with this?

---
When running a production system there's an endless stream of issues that have the potential to cause you significant hassle. How should you deal with this?

The list of potential ways your production system can come tumbling down is endless: lightning strike, coffee spill, running out of file descriptors, security vulnerability of the day, new behaviour in the latest iOS update, escalating machine costs, failure to scale, missing monitoring, and that's not even considering bugs in your code!

It's easy to get overwhelmed by these interrupts as they stream in. If you deal with each on an emergency basis you'll be in a panic most of the time, and ultimately you and your team will get burned out.

To avoid getting fatigued, a better approach is to step back and have a framework in place to deal with potentially critical interrupts. Rank each by severity and likelihood, and then prioritise against each other and all the other work on your plate. A good monitoring system that can alert to let you know when you're close to key limits is also useful. Rather than continuously fretting and checking to see if you're close to a limit, it can notify you in plenty of time to handle it.

Balance is important. Just as you shouldn't let your long term project work take the back seat to surprises, neither should project work hold up essential patching.
