---
layout: post
title: Common Misperceptions about Search as a Service
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

Since the first SaaS IPO by [salesforce.com](http://www.salesforce.com/), the
SaaS (Software as a Service) model has boomed in the last decade to become a
global market that is worth billions today. It has taken a long way and a lot
of evangelisation to get there.

Before [salesforce.com](http://www.salesforce.com/) and the other SaaS
pioneers succeeded at making SaaS a standard model, the IT departments were
clear: the infrastructure as well as the whole stack had to be behind their
walls. Since then, mindsets have shifted with the cloud revolution, and you
can now find several softwares such as Box, Jive or Workday used by a lot of
Fortune 500 companies and millions of SMBs and startups.

Everything is now going SaaS, even core product components such as internal
search. This new generation of SaaS products is facing the same misperceptions
their peers faced years ago. So today, we wanted to dig into the
misperceptions about search as a service in general.

## Hosting your search is way more complex and expensive than you may think

Some people prefer to go on-premises as they only pay for the raw resource,
especially if they choose to run open source software on it. By doing this,
they believe they can skip the margin layer in the price of the SaaS
solutions. The problem is that this view highly under-estimates the Total Cost
of Ownership (TCO) of the final solution.

Here are some reasons why hosting your own search engine can get extremely
complex & expensive:

### Hardware selection

A search engine has the particularity of being very IO (indexing), RAM
(search) and CPU (indexing + search) intensive. If you want to host it
yourself, you need to make sure your hardware is well sized for the kind of
search you will be handling. We often see companies that run on under-sized
EC2 instances to host their search engine are simply unable to add more
resource-consuming features (faceting, spellchecking, auto-completion).
Selecting the right instance is more difficult than it seems, and you'll need
to review your copy if your dataset, feature list or queries per second (QPS)
change. Elasticity is not only about adding more servers, but is also about
being able to add end-users features. Each Algolia cluster is backed by 3
high-end bare metal servers with at least the following hardware
configuration:

  * **CPU**: Intel Xeon (E5-1650v2) 6c/12t 3,5 GHz+/3,9 GHz+
  * **RAM**: 128GB DDR3 ECC 1600MHz
  * **Disk**:  1.2TB  SSD (via 3 or 4 high-durability SSD disks in RAID-0)

This configuration is key to provide instant and realtime search, answering
queries in <10ms.

### Server configuration

It is a general perception of many technical people that server configuration
is easy: after all it should just be a matter of selecting the right EC2
Amazon Machine Image (AMI) + a puppet/chef configuration, right?
Unfortunately, this isn't the case for a search engine. Nearly all AMIs
contain standard kernel settings that are okay if you have low traffic, but a
**nightmare** as soon as your traffic gets heavier. We've been working with
search engines for the last 10 years, and we still discover kernel/hardware
corner cases every month! To give you a taste of some heavyweight issues
you'll encounter, check out the following bullet points:

  * **IO**: Default kernel settings are **NOT** optimized for SSDs!!! For example, Linux's I/O scheduler is configured to merge some I/Os to reduce the hard-drive latency while seeking the disk sectors: non-sense on SSD and slowing the overall server performance.
  * **Memory**: The kernel caches a lot, and that's cool... most of the time. When you write data on the disk, it will actually be written in the RAM and flushed to disk later by the pdflush process. There are some advanced kernel parameters that allow configuration. vm.dirty_background_ratio is one of them: it configures the maximum percentage of memory that can be "dirty" (in cache) before it is written on the disk.  In other words, if you have 128GB of RAM, and you are using the default value of 10% for dirty_background_ratio, the system will only flush the cache when it reaches 12GB!!!! Flushing such bursts of writes will **slow down your entire system** (even on SSD), killing the speed of all searches & reads. [Read more](http://lonesysadmin.net/2013/12/22/better-linux-disk-caching-performance-vm-dirty_ratio/).
  * **Network**:  When calling the listen function in BSD and POSIX sockets, an argument called the backlog is accepted. The backlog argument defines the maximum length of the queue of pending connections for sockfd. If the backlog argument is higher than the value in net.core.somaxconn, it is silently truncated to that value. The default value is 128 which is **way too low**! If a connection request arrives when the queue is full, the client may receive an error with an indication of ECONNREFUSED. [Read more](http://engineering.chartbeat.com/2014/01/02/part-1-lessons-learned-tuning-tcp-and-nginx-in-ec2/) & [even more](https://www.youtube.com/watch?v=yL4Q7D4ynxU).

We've been working hard to fine-tune such settings and it has allowed us to
handle today several thousands of search operations per second on one server.

### Deployment & upgrades are complex

Upgrading software is one of the main reasons of service outages. It should be
fully automated and capable of rolling back in case of a deployment failure.
If you want to have a safe deployment, you would also need a pre-production
setup that duplicates your production's setup to validate a new deployment, as
well as an A/B test with a part of your traffic. Obviously, such setup
requires additional servers. At Algolia, we have test and pre-production
servers allowing us to validate every deployment before upgrading your
production cluster. Each time a feature is added or a bug is fixed on the
engine, all of our clusters are updated so that everyone benefits from the
upgrade.

### Toolbox vs features

On-premises solutions were not built to be exposed as a public service: you
always need to build extra layers on top of it. And even if these solutions
have plenty of APIs and low-level features, turning them into end-user
features requires time, resources and a lot of engineering (more than just a
full-stack developer!). You may need to re-develop:

  * ****Auto-completion**:** to suggest best products/queries directly from the search bar while handling security & business filters (not only suggesting popular entries);
  * **Instant-Faceting:** to provide realtime faceting refreshed at each keystroke;
  * ****Multi-datacenter replication**:** synchronize your data across multiple instances and route the queries to the right datacenter to ensure the best search performance all around the world;
  * **Queries analytics**: to get valuable information on what and how people search;
  * **Monitoring**: To track in realtime the state of your servers, the storage you use, the available memory, the performance of your service, etc.

## On-premises is not as secure as one might think

Securing a search engine is very complex and if you chose to do it yourself,
you will face three main challenges:

  1.  **Controlling who can access your data**: You probably have a model that requires permissions associated with your content. Search as a service providers offer packaged features to handle user based restrictions. For example you can generate an API Key that can only target specific indexes. Most on-premise search engines do not provide any access control feature.
  2. **Protecting yourself against attacks**: There are various attacks that your service can suffer from (denial of service, buffer overflow, access control weakness, code injection, etc.). API SaaS providers put a lot of effort into having the best possible security. For example API providers reacted the most quickly to the "HeartBleed" SSL vulnerability; It only took a few hours after disclosure for [Twilio](https://www.twilio.com/blog/2014/04/customer-security-notice-on-cve-2014-0160-heartbleed-disclosure.html), [Firebase](https://www.firebase.com/blog/2014-04-08-open-ssl-security-update.html) and [Algolia](http://blog.algolia.com/dealing-openssl-security-issue/) to fix the issue.
  3. **Protecting yourself from unwarranted downloads:** The search feature of your website can easily expose a way to grab all your data. Search as a service providers offer packaged features to help prevent this problem (rate limit, time-limited API Key, user-restricted API Key, etc.).

Mastering these three areas is difficult, and API providers are challenged
every day by their customers to provide a state-of-the-art level of security
in all of them. Reaching the same level of security with an on-premise
solution would simply require too much investment.

## Search as a service is not reserved to simple use cases

People tend to believe that search as a service is only good for basic use
cases, which prevents developers from implementing fully featured search
experiences. The fact of the matter is that search as a service simply handles
all of the heavy lifting while keeping the flexibility to easily configure the
engine. Therefore it enables any developers, even front-end only developers,
to build complex instant search implementation with filters, faceting or geo-
search. For instance, feel free to take a look at
[JadoPado](http://jadopado.com), a customer who developed a fully featured
instant search for their e-commerce store. Because your solution runs inside
your walls once in production,  you will need a dedicated team to constantly
track and fix the multiple issues you will encounter. Who would think of
having a team dedicated to ensuring their CRM software works fine? It makes no
sense if you use a SaaS software like most people do today. Why should it make
more sense for components such as search? All the heavy lifting and the
operational costs are now concentrated in the SaaS providers' hands, making it
eventually way more cost-efficient for you..

