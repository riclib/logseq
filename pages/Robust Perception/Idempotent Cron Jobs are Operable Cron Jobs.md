---
created: [[Aug 4th, 2021]]
type: #clipping
tags: prometheus 
source: https://www.robustperception.io/idempotent-cron-jobs-are-operable-cron-jobs
author: [[Brian Brazil]] 
---
> Having to reconstruct how far a failed cron job had gotten and what exact parameters it was run with can be error prone and time consuming. There is a better way.

# Idempotent Cron Jobs are Operable Cron Jobs


Having to reconstruct how far a failed cron job had gotten and what exact parameters it was run with can be error prone and time consuming. There is a better way.

I assume I'm not the only one who at some point or other has been woken up due to a single run of a cron job failing, and having to piece together the exact commands needed to re-run it correctly. This was usually due to the times and dates the cron job was working over being determined on the fly, and thus different for every run. So for example one run might be run to process only yesterday's data.

A better way would be to pass in the day/time you want to be processed up to, and then have the cron job figure out where it left off and continue from there. For example after processing an hour's worth of data the job could write out a small checkpoint indicating that it's done with that hour. On startup the cron job could read the last checkpoint, and continue on from there. If there is a failure, you can simply re-run the job without having to worry about the command line arguments. This is known as [idempotence](https://en.wikipedia.org/wiki/Idempotence), that multiple runs of a job result in the same state as a single run.

This has additional benefits beyond making it easy to manually recover from a failure. You can also use idempotency to avoid being alerted in the first place, after all a single failure of a cron job should in principle never cause a human to be paged nor require human intervention. To do this you can setup the cron job to run at least twice the frequency that you actually need it, so for example a daily job might be run twice a day instead. That way if there is a failure it'll automatically retry, and if it worked the first time then the second run will see that everything is okay and terminate promptly.

This brings us to monitoring. With this setup you no longer have to care about individual runs, instead only that the cron job completed successfully recently enough. So for example for a cronjob you need to succeed daily, as long as it had succeeded in the last (day + maximum likely runtime) then you're good. Having the cron job report the timestamp at which it last succeeded via the [textfile collector](https://www.robustperception.io/using-the-textfile-collector-from-a-shell-script) or [Pushgateway](https://www.robustperception.io/monitoring-batch-jobs-in-python) allows you to create alerts like

groups:
- name: test.rules
  rules:
  - alert: MyBatchJobNoRecentSuccess
    expr: time() - mybatchjob\_last\_success{job="my\_batch\_job"} > 3600 \* 26
    annotations:
      description: mybatchjob last succeeded {{humanizeDuration $value}} ago

which would alert if it hasn't run in the succeeded 26 hours.

One small tip with such alerts. You know when jobs usually complete, and thus when a succession of failures would be alerted on. It is wise to tune the threshold so this occurs during working hours, rather than in the middle of the night. By making a page go off at 10am rather than 7am it not only helps preserve sleep, but it's more likely that you'll take appropriate action for the cost of a delay that shouldn't be relevant for a daily alert.

_Want to reduce alert noise? [Contact us](mailto:prometheus@robustperception.io)._

Published by [[Brian Brazil]] in [Posts](https://www.robustperception.io/category/posts)
