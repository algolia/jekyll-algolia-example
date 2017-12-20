---
layout: post
title: Welcome Texas!
author:
  login: gaetan
  email: gaetan@algolia.com
  display_name: gaetan
  first_name: Gaetan
  last_name: Gachet
---

<p class="deprecation-notice">
You probably already know it: any millisecond that end-users have to wait to
get their results drives us nuts. But what on Earth does this have to do with
Texas? Actually a lot!
</p>

## You want your search to be instant? Let's talk network...

When looking at the speed of search on a website or a mobile application, the
performance of the search engine is just one part of the equation. When you're
using an extremely fast engine, network latency and saturated links quickly
become your biggest enemies: it simply takes time for the user query to reach
the engine and for the results to get back to the user's browser.

In some cases, the round trip can easily take more than a second. In the US, it
can take up to 300ms to simply establish an SSL connection between the two
coasts.  All this also applies to the communications between your backend and
the servers that host your search engine. The network can simply ruin the real
time experience you hoped to offer with your search engine.

## A new US Central point of presence to reach a 25ms total delivery time across the US

A great search experience is to drive end-users towards what they're looking
as quickly and seamlessly as possible. For us at Algolia it means to be able
to dynamically update the content displayed as the end-user is typing a query.
Being able to offer this find as-you-type experience obviously requires a very
performant search engine but it also requires to host the search engine itself
as close as possible to the end-user in order to tackle the network latency.

This is why we are adding this new US Central region to our existing twelve
regions. With the addition of the Dallas PoP, Algolia's API is now accessible
from thirteen different regions including US (East, West and Central),
Australia, Brazil, Canada, France, Germany, Hong Kong, India, Japan, Russia,
and Singapore.

If your audience is spread out across multiple regions, you can use Algolia
from a combination of these regions to ensure minimal results delivery time
and optimal speed for all your users (Algolia's Distributed Search Network
automatically routes user queries to your closest region).

This new US Central PoP, combined with Algolia's US East and US West PoPs, now
allows to deliver search results across the US with less than 25 milliseconds of
latency. This guarantees a seamless find-as-you-type experience on websites and
mobile applications all across the US.

[![dallas2][1]](https://blog.algolia.com/wp-content/uploads/2015/07/dallas2.jpg)

## Getting closer to additional infrastructure providers

When you choose SaaS providers, especially when their service becomes a core
component of your product, you probably prefer the ones hosted close to where
you operate your backend, for latency and availability reasons. This is
actually why we initially started in the US by opening PoPs in Ashburn (VA)
and San Jose (CA), close to the AWS PoPs, which most of our customers rely on
today.

Our new presence in Texas allows services which rely for their backend on
local infrastructure providers such as Rackspace and Softlayer to also benefit
from the full power of Algolia. This new PoP offers them an extremely low
network latency between their backend and our API.

If you're not already an Algolia user and you want to give it a try, simply
[sign up][2] for a 14 day trial and select
the US Central region in the process.

If you are already using Algolia and want to migrate to the US Central region,
simply drop us a line at [support@algolia.com][3] or
on the live chat.

If you're none of the two above, we still think you're awesome!

Cheers!


[1]: /algoliasearch-jekyll-hyde/assets/dallas2.jpg
[2]: https://www.algolia.com/users/sign_up
[3]: mailto:support@algolia.com
