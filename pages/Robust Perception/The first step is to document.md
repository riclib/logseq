---
created: [[Aug 8th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/the-first-step-is-to-document
author: [[Brian Brazil]] 
---
# The first step is to document

> When you've a complicated manual process that you want to improve, your first instinct as a developer might be to jump in and start coding. Hold off a bit, the first step is to document.

---
When you've a complicated manual process that you want to improve, your first instinct as a developer might be to jump in and start coding. Hold off a bit, the first step is to document.

By rushing in you may miss the forest for the trees. The first time around many steps are likely to be fuzzy and require a human to judge the exact steps in each case. Attempting to handle all the potential cases as you find them will likely lead to [yak shaving](http://sethgodin.typepad.com/seths_blog/2005/03/dont_shave_that.html) and you'll be left frustrated with a half-implemented solution.

[![](http://www.robustperception.io/wp-content/uploads/2016/03/automation.png)](https://xkcd.com/1319/)

Obligatory XKCD comic

## Grow a document

To avoid this, the next time you have to perform this process open up an empty document (on a wiki, word processor, online document, text editor or whatever) and write down the steps as you perform them. At this first stage you should include enough detail that someone else familiar with the systems in question could perform them (e.g. they know all the naming and procedural conventions), without going into painstaking detail. This gives you an outline of what's involved.

The next few times you need to do the process, use the document as your base. Expand on instructions so that there's less implied logic, and explain any corner cases that come up. Having someone else, particularly a new employee, to follow and improve the document is beneficial.

## You've got a document, now what?

By now you've got a reasonably good description of what the process is that anyone on the team could follow. This in itself is a benefit as you've reduced your [bus factor](https://en.wikipedia.org/wiki/Bus_factor), and you're less likely to forget or muddle a step in future.

In terms of making the process easier, it should by now be obvious which steps are particularly repetitive and would benefit from scripting or automation. By focusing on these low hanging fruit you can get immediate gains. Fully automating every step, particularly for tasks you rarely perform, isn't always a [good return on your time](https://xkcd.com/1205/).

There's another potential outcome here too. When you've several such documents for various processes, you may notice certain common themes among the steps that need to be performed. This is a hint that there is infrastructure that you might be missing, such as [configuration management](http://www.robustperception.io/do-you-have-basic-infrastructure/) or a central user database.

By starting with documentation first you gain value sooner, and it helps you to spot bigger opportunities for improvement.

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
