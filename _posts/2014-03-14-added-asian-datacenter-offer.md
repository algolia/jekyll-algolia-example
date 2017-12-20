---
layout: post
title: Algolia Now Provides Realtime Search in Asia!
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

[![New datacenter allows realtime search in Asia][1]](http://blog.algolia.com/added-asian-
datacenter-offer/screen-shot-2014-03-13-at-17-51-50/)

One of the terrific advantages of building a SaaS company is that your clients
can be anywhere in the world. We now have customers in more than 15 different
countries distributed across South America, Europe, Africa, and, of course,
North America. We feel incredibly lucky to have so many international
customers trusting us with their search.

Language support is one of the key factors that enabled us to enter these
markets. Since the beginning, we wanted to support every language used on the
Internet. To back our vision with action, we developed a very good support of
Asian languages over time. As an example, we are able to automatically
retrieve results in Traditional Chinese when the query is in Simplified
Chinese (or vice-versa). You simply need to add objects in Chinese, Japanese
or Korean, and we handle the language processing for you.

Despite the fact that we could process Asian languages well, we didn't plan to
open an Asian datacenter so early, mainly because we thought the API as a
service market was less mature in Asia than in the US or Europe. But we were
surprised when an article on [36kr.com][2]
gave us dozen of signups from China. We got more signups from China in the
past month than from Canada!

One of our core values is the speed of our search engine. To provide a
realtime search experience, we want the response times to be lower than 100ms,
including the round trip to search servers. In this context a low latency is
essential. Up to now we have been able to cover North America and Europe in
less than 100ms (search computation included) but our latency with Asia was
between 200ms and 300ms.

The first step of our on-boarding process is to select the datacenter where
your search engine is hosted (we offer multi-datacenter distribution only for
enterprise users). Interestingly, we discovered that we had no drop for
European & US users but it became significant for others. It was a difficult
choice for people outside of these two regions, or even between the two
datacenters. So we also now display the latency from your browser and pre-
select the "closest" datacenter.

To propose better latency and to reduce friction in the on-boarding process,
it was clear that we had to add a datacenter in Asia. We chose Singapore for
its central location. Unfortunately, the hosting market is very different in
Asia. It's much more expensive to rent servers, so we sadly had to add a
premium on plan prices when choosing this datacenter.

We are very happy to open this new datacenter in Asia with a latency that
reaches our quality standard. Now that Algolia provides realtime search in
Asia, we are even happier to be able to help multinational websites and apps
provide a great search experience to all their users across Europe, North
America & Asia in less than 100ms with our multi-datacenter support!*

_Multi-datacenter support is currently only available for Enterprise
accounts._


[1]: /algoliasearch-jekyll-hyde/assets/Screen-Shot-2014-03-13-at-17.51.50-300x199.png
[2]: http://www.36kr.com/p/209747.html
