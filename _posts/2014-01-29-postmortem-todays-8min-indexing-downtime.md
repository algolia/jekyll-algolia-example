---
layout: post
title: Postmortem of today's 8min indexing downtime
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

Today (Jan 29) at 9:30pm UTC, our service experienced an 8 minute partial
outage during which we have rejected many write operations sent to the
indexing API (exactly 2841 calls). We call it "partial" as all search queries
have been honored without any problem. For end-users, there was no visible
problem.

Transparency is in our DNA: this outage is visible on our status page
([status.algolia.com][1]) but we also wanted to share
with you all the details of the outage and more importantly the details of our
response.

## The alert

This morning I fixed a rare bug in indexing complex hierarchical objects. This
fix successfully passed all the tests after development. We have 6000+ unit
tests and asserts, and 200+ non regression tests. So I felt confident when I
entered the deploy password in our automatic deployment script.

A few seconds after, I started to receive a lot of text messages on my
cellphone.

We developed several embedded probes to detect all kinds of problems and alert
us using Twilio and Hipchat APIs. They detect for example:

  * a process that restart
  * an unusually long query
  * a write failure
  * a low memory warning
  * a low disk-free warning
  * etc.

In case embedded probes can't run, other external probes run once a minute
from an independent datacenter (Google App Engine). These also automatically
update our status page when a problem impacts the quality of service.

Our indexing processes were crash looping. I immediately decided to rollback
to the previous version.

## The rollback

Until today, our standard rollback process was to revert the commit, launch
the recompile and finally deploy. This is long, very long when your know that
you have an outage in production. The rollback took about 5 minutes in total
out of the 8 minutes.

## How we will avoid this situation in the future

Even if the outage was on a relatively small period of time, we still believe
it was too long. To make sure this will not happen again:

  * We have added a very fast rollback process in the way of a simple press button like the one we use to deploy. An automatic deploy is nice, but an automatic rollback is actually more critical when needed!
  * Starting now, we will deploy new versions of the service on clusters hosting community projects such as Hacker News Search or Twitter handle search, before pushing the update on clusters hosting paying customers. Having real traffic is key to detect some types of errors. Unit-tests & non-regression tests cannot catch everything.
  * And of course we added non-regression tests for this specific error.

## Conclusion

Having all these probes in our infrastructure was key to detect today's
problem and react quickly. In real conditions, it proved not to be enough. In
a few hours we have implemented a much better way to handle this kind of
situation. The quality of our service is our top priority. Thank you for your
support!


[1]: http://status.algolia.com
