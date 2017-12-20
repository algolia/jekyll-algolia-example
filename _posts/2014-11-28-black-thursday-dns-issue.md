---
layout: post
title: Black Thursday DNS issue
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

Today we had a severe DNS issue that impacted some of our users during a total
of 5 hours. Although most of our customers were not impacted, for some of them
search simply went down. This event and its details deserved to be fully
disclosed.

## The context

Up until recently, we were using Amazon Route 53 for our DNS routing needs.
When we started to design our Distributed Search Network
([algolia.com/dsn][1] a few months ago, we quickly
realized that our needs were out of Route 53's scope: we needed a custom
routing per user and the two options of Route 53 simply didn't work:

  * Latency-based routing is limited to the 9 regions of AWS and we have 12;
  * With geography-based routing you need to indicate country per country how you want to resolve the IP.

This is a tedious process for a not even good solution as route 53 does not
support EDNS right now.

So we started to look for new DNS options. Choosing the best DNS provider is
not something you do overnight. It took us months to benchmark several vendors
and find the right one: NSOne. The [filter chain feature of
NSOne][2] was a
perfect fit for our use case and the NSOne team was great in understanding our
needs and even went the extra mile for us by building a specific module,
allowing better performance.

Something we also discovered during this benchmark was that the algolia.io
domain was not good for performance compared to algolia.net, as there are far
more DNS servers in the .net anycast network than in the .io one. The NSOne
team offered us a smart solution based on linked domain, so we wouldn't have
to maintain two zones ourselves.

## The migration

The goal of the migration was to move from Route 53 to NSOne. For several
weeks we have been working on importing the records in NSOne and making sure
Route 53 and NSOne were synchronized. Our initial tests revealed some issues
but after a few days of continuous updates without any difference between
Route 53 and NSOne, we started to be confident about our synchronization and
started the migration of the [demos of our
website][3] to make them target the new
algolia.net domain. We tested the performance and resolution from all NSOne
POP ([https://nsone.net/technology/network/](https://nsone.net/technology/netw
ork/)) to be sure there were no glitches.

These first production tests were successful, synchronization was ok,
performance and routing were good, so we decided to move the .io domain from
Route 53 to NSOne as well.

## The D-day

The big issue when changing the DNS is that it is global and involves caching
logics, making rollbacking complex. With users in 45 countries it is almost
impossible to find a suitable time for everyone: DNS changes cannot be done
gradually. We decided to push the update during the night for the US, at 4am
EST.

We witnessed a quickly rising number of queries targeting NSOne and it's once
we reached about 1,000 DNS queries per second that we started to receive our
first complain about failed DNS resolution. This routing issue was not
impacting all DNS resolutions but some of them were replying with a NXDOMAIN
answer, the equivalent of a DNS "404 not found".

```
$ dig APPID-1.algolia.io`
; <<>> DiG 9.9.5-4.3-Ubuntu <<>> APPID-1.algolia.io
;; global options: +cmd
;; Got answer:
;; ->>HEADER< ;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1
;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4096
;; QUESTION SECTION:
;APPID-1.algolia.io. IN A
;; AUTHORITY SECTION:
algolia.io. 45 IN SOA dns1.p03.nsone.net. hostmaster.nsone.net.
1414873854 43200 7200 1209600 3600
;; Query time: 24 msec
;; SERVER: 213.133.100.100#53(213.133.100.100)
;; WHEN: Thu Nov 27 10:49:33 CET 2014
;; MSG SIZE rcvd: 115
```

After double-checking our DNS zone for those specific records, we understood
it was a NSOne bug related to our custom routing code. We immediately
rollbacked to Route 53.

The NSOne support was really quick to react and they found the issue pretty
quickly: the issue only concerned some DNS with EDNS support on the algolia.io
domain. The algolia.net domain was not impacted, explaining why all the tests
we've done weren't able to detect the issue before.

Unfortunately, it did not stop here and something very unexpected happened:
some customers (even not priorly impacted) started to face issues right after
the rollback to Route 53.

In order to improve performance, the custom Algolia module developed by NSOne
was doing some translation on our records: APPID-1.algolia.io is translated
into 1.APPID.algolia.io and then resolved to CNAME for the actual server in
the cluster serving that customer. The translation of APPID-1.algolia.io to
1.APPID.algolia.io was done with a TTL of 86400 seconds (1 day). Since these
zones did not exist in Route 53 before, it was not possible to resolve there
records anymore. What made the situation even worse was the TTL far exceeding
the TTL of NS records. Most of the DNS servers flushed their cache for the
domain, once the nameservers changed. But the remaining ones kept the record
cached.

**TL;DR: Do not forget about IPv6.** As if it was not enough, we eventually discovered something else: our custom DNS module was resolving APPID-X.algolia.io to X.APPID.algolia.io only in a case that there were no direct resolutions to an IP address. This translation worked pretty well as we had all the A records set. But some customers started to report weird resolutions. Normally we resolve APPID-1.algolia.io -> 1.APPID.algolia.io -> servername-1.algolia.io -> IP. Which was completely fine until the moment an IPv6 AAAA request came. Since we did not have AAAA records, the custom filter started to resolve: APPID-1.algolia.io -> 1. APPID.algolia.io -> servername-1.algolia.io -> 1.servername.algolia.io -> nothing.

We were in a bad situation feared by all engineers, this lonely moment when
you really miss a "purge cache" feature.

Eventually, as soon as we got confirmation of the fix by NSOne, we changed
again the DNS of algolia.io to NSOne and helped our customers to workaround
the issue before the cache expiration:

for our customers impacted by the NXDOMAIN issue, a simple migration to the
algolia.net domain instead of the algolia.io problem fixed the issue;

for those impacted by the Route 53 rollback issue, we created new DNS records
for them to avoid work-around DNS caches.

## Conclusion: what we learned

This is by far the biggest problem we have encountered since the launch of
Algolia. Although the first issue was almost impossible to anticipate, we have
made mistakes and should have handled a few things differently:

DNS is a highly critical component and being the first to use an external
custom module was not a good idea, even if it improved performance;

Putting more thought into the rollback part of our deployment would have
helped us anticipate the second issue. For a component as critical as a DNS,
having a robust rollback process is mandatory, no matter how much work it
represents and even though such an event is extremely unlikely to happen.

We're very sorry for this disruption. We wanted to share these technical
details to shed some light on what happened and what we've done in response.
Thanks for your patience and support.

If you think we missed anything or if you'd like to share your advice on your
own best practices, your comments are really welcome.


[1]: https://www.algolia.com/dsn)
[2]: https://nsone.net/2014/03/nsone-filter-chain-quick-primer/
[3]: https://www.algolia.com/demos
