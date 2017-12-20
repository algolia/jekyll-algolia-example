---
layout: post
title: Don't let network latency ruin the search experience of your international users
author:
  login: marie-auxille
  email: marie-auxille.denis@algolia.com
  display_name: Marie-Auxille Denis
  first_name: Marie-Auxille
  last_name: Denis
---

At Algolia, we allow developers to provide a unique interactive search
experience with as-you-type search, instant faceting, mobile geo-search and on
the fly spell check.

Our Distributed Search Network aims at removing the impact of network latency
on the speed of search, allowing our customers to offer this instant
experience to all their end-users, wherever they may be.

## Every millisecond matters

We are obsessed with speed and we're not the only ones: Amazon found out that
100ms in added latency cost them 1% in sales. [The lack of responsiveness for
a search engine can really be damaging for one's
business][1]. As
individuals, we are all spoiled when it comes to our search expectations:
Google has conditioned the whole planet to expect instant results from
anywhere around the world.

![user-experience][2]

We have the exact same expectations with any online service we use. The thing
is that for anyone who is not Google, **it is just impossible to meet these
expectations** because of the network latency due to the physical distance
between the service backend that hosts the search engine and the location of
the end-user.

Even with the fastest search engine in the world, it can still take hundreds
of milliseconds for a search result to reach Sydney from San Francisco. And
this is without counting the bandwidth limitations of a saturated oversea
fiber!

## How we beat the speed of light

Algolia's Distributed Search Network (DSN) removes the latency from the speed
equation by **replicating your indices to different regions around the world,
where your users are**.

Your local search engines are clones synchronized across the world. DSN allows
you to **distribute your search among 12 locations** including the US,
Australia, Brazil, Canada, France, Germany, Hong Kong, India, Japan, Russia,
and Singapore. Thanks to our 12 data centers, your search engine can now
**deliver search results under 50ms in the world's top markets**, ensuring an
optimal experience for all your users.

![DSN-b3ce122c790c492c2f2c8ddbabaae464][3]

## How you activate DSN

Today, DSN is only accessible to our Starter, Growth, Pro and Enterprise plan
customers. To activate it, you simply need to go in the **"Region" tab at the
top of your Algolia dashboard and select "Setup DSN"**.

[![shot][4]](https://www.algolia.com/dsn/setup)

You will then be displayed with a map and a selection of your top countries in
terms of search traffic. Just **select our DSN data centers on the map and see
how performance in those countries is optimized**.

Algolia will then automatically take care of the distribution and the
synchronization of your indices around the world. End-users' queries will be
automatically routed to the closest data center among those you've selected,
ensuring the best possible experience. Algolia DSN delivers an ultra low
response time and automatic fail-over on another region if a region is down.

[![][5]](https://www.algolia.com/dsn/setup)

It is that simple!

Today, several services including HackerNews, TeeSpring, Product Hunt and
Zendesk are leveraging **Algolia DSN to provide faster search to their global
users**.

Want to find out more about the Algolia experience ?

[Discover and try it here][6]!


[1]: http://glinden.blogspot.fr/2006/11/marissa-mayer-at-web-20.html
[2]: /algoliasearch-jekyll-hyde/assets/user-experience.jpg
[3]: /algoliasearch-jekyll-hyde/assets/DSN-b3ce122c790c492c2f2c8ddbabaae464.jpg
[4]: /algoliasearch-jekyll-hyde/assets/shot.jpg
[5]: /algoliasearch-jekyll-hyde/assets/dsn-shot.jpg
[6]: https://www.algolia.com/features
