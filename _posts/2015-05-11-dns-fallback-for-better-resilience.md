---
layout: post
title: DNS fallback for better resilience
author:
  login: adam
  email: adam@algolia.com
  display_name: Adam Surak
  first_name: Adam
  last_name: Surak
---

At Algolia, we are obsessed with finding a way to have a 99.9999% available
architecture. On our way to achieve that, we have to make sure every piece of
the architecture can safely fail without affecting the service.

The first point of the architecture where a customer's request starts to
interact with our service is not the router in the datacenter, but a DNS
resolving a domain name to the IP address "long time" before that. This piece
of architecture is very often overlooked and that is no surprise as you mostly
get best-effort DNS service automatically with your server.

## Latency

For couple months we are a happy user of [NSONE][1] that
provides us with the first level of logic. We use NSONE for its superb
performance and data-driven DNS that gives us control in steering the traffic
of our [Distributed Search Network][2] to the proper
server - whether it means closest or simply available one. But as any other
network dependent service, there are factors outside of NSONE's control that
can influence availability of its DNS resolves and consequently Algolia. BGP
routing is still a huge magic and "optimizations" of some ISPs are beyond
understanding. Well, they do not always make the optimizations in the
direction we would like to. For some services the change of DNS resolution
time from 10 to 500ms does not mean a lot but for us it is a deal breaker.

[![nsone-dig-latency][3]](https://blog.algolia.com/wp-content/uploads/2015/05/nsone-dig-latency.png) Resolution of latency-1 via NSONE

## DDoS

When we started to think about our DNS dependency, we remembered the [2014
DDoS attack on UltraDNS][4] and the situation when there was not enough
[#hugops][5] for all the services impacted.
During the previous [attack on UltraDNS in 2009][6] even
big names like Amazon and SalesForce got impacted.

## Solution

In most of the cases it would mean adding another DNS name server from a
different provider and replicate the records. But not in ours. NSONE has some
unique features that we would have to give up and find a common feature subset
with a different provider. In the end we would have to serve a portion of DNS
resolutions via slower provider for no good reason.

Since we provide custom made API clients we have one more place where to put
additional logic. Now came a time to choose a resilient provider for our
secondary DNS and since we like AWS, Route53 was a clear choice. Route53 has
ok performance, many POPs around the world and API we already had integration
for.

In the last moment, one more paranoid idea came to us - let's not rely on a
single [TLD][7]. No good reason
for that, it was just "what if...?" moment.

[![route53-dig-latency][8]](https://blog.algolia.com/wp-content/uploads/2015/05/route53-dig-latency.png) Resolution of latency-1 via
Route53

Right now, all the latest versions of our API clients (detailed list below)
use multiple domain names. "algolia.net" is served by NSONE and provides all
the speed and intelligence, "algolianet.com" is served by Route53 in case that
for any reason contacting server via "algolia.net" fails. It brings more work
to our side, brings more cost on our side but it also brings better sleep for
our customers, their customers and us.

And now we can think what else can fail...

Minimal versions of API clients with support of multiple DNS:

  * [Javascript v2: 2.9.6](https://github.com/algolia/algoliasearch-client-js/releases/tag/2.9.6)
  * [Javascript v3: 3.1.0](https://github.com/algolia/algoliasearch-client-js)
  * [Node.js: 1.8.0](https://github.com/algolia/algoliasearch-client-node)
  * [Ruby: 1.4.1](https://github.com/algolia/algoliasearch-client-ruby)
  * [Ruby on rails: Ruby dependency 1.4.1](https://github.com/algolia/algoliasearch-rails)
  * [Python: 1.5.2](https://github.com/algolia/algoliasearch-client-python)
  * [PHP: 1.5.5](https://github.com/algolia/algoliasearch-client-php)
  * [Java: 1.3.5](https://github.com/algolia/algoliasearch-client-java)
  * [Android: 1.6.3](https://github.com/algolia/algoliasearch-client-android)
  * [Objective-C: 3.4.1](https://github.com/algolia/algoliasearch-client-objc)
  * [C-Sharp: 3.1.0](https://github.com/algolia/algoliasearch-client-csharp)
  * [Go: 1.2.0](https://github.com/algolia/algoliasearch-client-go)


[1]: https://nsone.net
[2]: https://www.algolia.com/dsn
[3]: /algoliasearch-jekyll-hyde/assets/nsone-dig-latency.png
[4]: https://threatpost.com/ultradns-dealing-with-ddos-attack/105806
[5]: https://twitter.com/hashtag/hugops
[6]: http://www.zdnet.com/article/ddos-attack-on-ultradns-affects-amazon-com-salesforce-com-petco-com/
[7]: http://en.wikipedia.org/wiki/Top-level_domain
[8]: /algoliasearch-jekyll-hyde/assets/route53-dig-latency.png
