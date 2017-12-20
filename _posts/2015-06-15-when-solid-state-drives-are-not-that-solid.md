---
layout: post
title: When Solid State Drives are not that solid
author:
  login: adam
  email: adam@algolia.com
  display_name: Adam Surak
  first_name: Adam
  last_name: Surak
---

It looked just like another page in the middle of the night. One of the
servers of our search API stopped processing the indexing jobs for an unknown
reason. Since we build systems in Algolia for high availability and
resiliency, nothing bad was happening. The new API calls were correctly
redirected to the rest of the [healthy machines in the
cluster](http://highscalability.com/blog/2015/3/9/the-architecture-of-
algolias-distributed-search-network.html) and the only impact on the service
was one woken-up engineer. It was time to find out what was going on.

UPDATE June 16:  
[A lot of
discussions](https://hn.algolia.com/story/9723066/when-solid-state-drives-are-not-that-solid)
started pointing out that the issue is related to the newly introduced queued
TRIM. This is not correct. The TRIM on our drives is un-queued and the issue we
have found is not related to the latest changes in the Linux Kernel to disable
this feature.

UPDATE June 17:  
We got contacted by Samsung and we provided them all the system specifications and all the information about the issue we had. We will continue to provide Samsung all the necessary information in order to resolve the issue.

UPDATE June 18:
We just had a conference call with the European branch and the Korean HQ of
Samsung. Their engineers are going to visit one of the datacenters we have
servers in and in cooperation with our server provider they will inspect the
mentioned SSDs in our SW and HW setup.

UPDATE June 19:  
On Monday June 22, the engineering team from Samsung is going analyze one of our servers in Singapore and if nothing is found on-site, the server will travel to Samsung HQ in Korea for further analysis.

UPDATE July 13:  
Since the last update of this blog-post, we have been in a cooperation with Samsung trying to help them find the issue, during this investigation we agreed with Samsung not to communicate until their approval.

As the issue was not reproduced on our server in Singapore, the reproduction
is now running under Samsung supervision in Korea, out of our environment.
Although Samsung requested multiple times an access to our software and
corrupted data, we could not provide it to them in order to protect the
privacy and data of our customers.

Samsung asked us to inform you about this:

  * Samsung tried to duplicate the failure with the latest script provided to them, but no single failure has been reproduced so far.
  * Samsung will do further tests, most likely from week 29 onwards, with a much more intensive script provided by Algolia.

After unsuccessful tries to reproduce the issue with Bash scripts we have
decided to help them by creating a small C++ program that simulates the
writing style and pattern of our application (no files are open with
O_DIRECT). We believe that if the issue is coming from a specific way we are
using the standard kernel calls, it might take a couple of days and terabytes
of data to be written to the drive. 

We have been informed by Samsung that no issue of this kind have been reported
to them. Our server provider has modified their Ubuntu 14.04 images to disable
the fstrim cron in order to avoid this issue. For the last couple of months
after not using trim anymore we have not seen the issue again.

UPDATE July 17:  
We have just finished a conference call with Samsung
considering the failure analysis of this issue. Samsung engineering team has
been able to successfully reproduce the issue with our latest provided binary.

 - Samsung had a concrete conclusion that the issue is not related to Samsung SSD or Algolia software but is related to the Linux kernel.
 - Samsung has developed a kernel patch to resolve this issue and the official statement with details will be released tomorrow, July 18 on Linux community with the Linux patch guide. Our testing code [is available on GitHub](https://github.com/algolia/trimtester).

This has been an amazing ride, thank you everyone for joining, we have arrived
at the destination.

* * *

The [NGINX](http://nginx.org/) daemon serving all the HTTP(S) communication of
our API was up and ready to serve the search queries but the indexing process
crashed. Since the indexing process is guarded by
[supervise](http://cr.yp.to/daemontools/supervise.html), crashing in a loop
would have been understandable but a complete crash was not. As it turned out
the filesystem was in a read-only mode. All right, let's assume it was a
cosmic ray :) the filesystem got fixed, files were restored from another
healthy server and everything looked fine again.

The next day another server ended with filesystem in read-only, two hours
after another one and then next hour another one. Something was going on.
After restoring the filesystem and the files, it was time for serious analysis
since this was not a one time thing. At this point, we did a breakdown of the
software involved in our storage stack and went through the recent changes.

## Investigation & debugging time!

We first asked ourselves if it could be related to our software. Are we using
non-safe system calls or processing the data in an unsafe way? Did we
incorrectly read and write the files in the memory before flushing it to disk?

  * Filesystem - Is there a bug in [ext4](https://en.wikipedia.org/wiki/Ext4)? Can we access the memory space of allocation tables by accident?
  * [Mdraid](https://raid.wiki.kernel.org/index.php/Linux_Raid) - Is there a bug in mdadm? Did we use an improper configuration?
  * Driver - Does the driver have a bug?
  * [SSD](https://en.wikipedia.org/wiki/Solid-state_drive) - Is the SSD dying? Or even worse, is there a problem with the firmware of the drive?

We even started to bet where the problem was and exactly proposed, in this
order, the possible solutions going from easy to super-hard.

Going through storage procedures of our software stack allowed us to set up
traps and in case the problem happens again, we would be able to better
isolate the corrupted parts. Looking at every single storage call of our
engine gave us enough confidence that the problem was not coming from the way
in which we manipulate the data. Unfortunately.

One hour later, another server was corrupted. This time we took it out of the
cluster and started to inspect it bit by bit. Before we fixed the filesystem,
we noticed that some pieces of our files were missing (zeroed) - file
modification date was unchanged, size was unchanged, just some parts were
filled with zeros. Small files were completely erased. 

This was weird, so we started to think if it was possible that our application
could access certain portions of the memory where the OS/filesystem had
something mapped because otherwise our application cannot modify a file without
the filesystem noticing. Having our software written in C++ brought a lot of
crazy ideas of what happened. This turned out to be a dead-end as all of these
memory blocks were out of our reach.

So is there an issue in the ext4? Going through the kernel changelog looking
for ext4 related issues was a terrifying experience. In almost every version
we found a fixed bug that could theoretically impact us. I have to admit, I
slept better before reading the changelog.

We had kernels 3.2, 3.10, 3.13 and 3.16 distributed between the most often
corrupted machines and waited to see which of the mines blows up. All of them
did. Another dead-end. Maybe there was an issue in ext4 that no one else has
seen before? The chance that we were this “lucky” was quite low and we did not
want to end up in a situation like that. The possibility of a bug in ext4 was
still open but highly improbable.

What if there was an issue in mdadm? Looking at the changelog gave us
confidence that we should not go down this path.

The level of despair was reaching a critical level and the pages in the middle
of the night were unstoppable. We spent a big portion of two weeks just
isolating machines as quickly as possible and restoring them as quickly as
possible. The one thing we did was to implement a check in our software that
looked for empty blocks in the index files, even when they were not used, and
alerted us in advance.

## Not a single day without corruptions

While more and more machines were dying, we had managed to automate the
restore procedure to a level we were comfortable with. At every failure, we
tried to look at different patterns of the corruption in hopes that we would
find the smallest common denominator. They all had the same characteristics.
But one thing started to be more and more clear - we saw the issue only on a
portion of our servers. 

The software stack was identical but the hardware was slightly different. Mainly
the SSDs were different but they were all from the same manufacturer. This was
very alarming and led us to contact our server provider to ask if they have ever
seen something like this before. It’s hard to convince a technical support
person about a problem that you see only once in a while, with the latest
firmware and that you cannot reproduce on demand.  We were not very successful
but at least we had one small victory on our side.

Knowing that the issue existed somewhere in the combination of the software
and drive itself, we reproduced the identical software stack from our servers
with different drives. And? Nothing, the corruption appeared again. So it was
quite safe to assume the problem was not in the software stack and was more
drive related. But what causes a block to change the content without the rest
of the system noticing? That would be a lot of rotten bits in a sequence...

The days started to become a routine - long shower, breakfast, restoring
corrupted servers, lunch, restoring corrupted servers, dinner, restoring
corrupted servers. Until one long morning shower full of thinking, “how big
was the sequence?” As it turned out, the lost data was always 512 bytes, which
is one block on the drive. 

One step further, a block ends up to be full of zeroes. A hardware bug? Or is
the block zeroed? What can zero the block?
[TRIM](https://en.wikipedia.org/wiki/Trim_(computing))! Trim instructs the SSD
drive to zero the empty blocks. But these block were not empty and other types
of SSDs were not impacted. We gave it a try and disabled TRIM across all of our
servers. It would explain everything!

The next day not a single server was corrupted, two days silence, then a week.
The nightmare was over! At least we thought so… a month after we isolated the
problem, a server restarted and came up with corrupted data but only from the
small files - including certificates. Even improper shutdown cannot cause
this.

Poking around in the source code of the kernel looking for the trim related
code, we came to the [trim blacklist](https://github.com/torvalds/linux/blob/e
64f638483a21105c7ce330d543fa1f1c35b5bc7/drivers/ata/libata-
core.c#L4109-L4286). This blacklist configures a specific behavior for certain
SSD drives and identifies the drives based on the regexp of the model name.
Our working SSDs were explicitly allowed full operation of the TRIM but some
of the SSDs of our affected manufacturer were limited. Our affected drives did
not match any pattern so they were implicitly allowed full operation.

## The complete picture

At this moment we finally got a complete picture of what was going on. The
system was issuing a TRIM to erase empty blocks, the command got
misinterpreted by the drive and the controller erased blocks it was not
supposed to. Therefore our files ended-up with 512 bytes of zeroes, files
smaller than 512 bytes were completely zeroed. When we were lucky enough, the
misbehaving TRIM hit the super-block of the filesystem and caused a
corruption. 

After disabling the TRIM, the live big files were no longer
corrupted but the small files that were once mapped to the memory and never
changed since then had two states - correct content in the memory and
corrupted one on the drive. Running a check on the files found nothing because
they were never fetched again from the drive and just silently read from the
memory. Massive reboot of servers came into play to restore the data
consistency but after many weeks of hunting a ghost we came to the end.

As a result, we informed our server provider about the affected SSDs and they
informed the manufacturer. Our new deployments were switched to different SSD
drives and we don't recommend anyone to use any SSD that is anyhow mentioned
in a bad way by the Linux kernel. Also be careful, even when you don't enable
the TRIM explicitly, at least since Ubuntu 14.04 the explicit
[FSTRIM](http://man7.org/linux/man-pages/man8/fstrim.8.html) runs in a cron
once per week on all partitions - the freeze of your storage for a couple of
seconds will be your smallest problem.

## TL;DR

### <del>Broken SSDs</del>: (Drives on which we have detected the issue)

  * SAMSUNG MZ7WD480HCGM-00003
  * SAMSUNG MZ7GE480HMHP-00003
  * SAMSUNG MZ7GE240HMGR-00003
  * Samsung SSD 840 PRO Series  
recently blacklisted for 8-series blacklist

  * Samsung SSD 850 PRO 512GB  
recently blacklisted as 850 Pro and later in 8-series blacklist

### <del>Working SSDs</del>: (Drives on which we have NOT detected the issue)

  * Intel S3500
  * Intel S3700
  * Intel S3710

