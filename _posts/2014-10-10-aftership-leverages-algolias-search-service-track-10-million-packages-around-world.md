---
layout: post
title: AfterShip Leverages Algolia's Search as a Service to Track 10 Million Packages Around The World
author:
  login: marie-auxille
  email: marie-auxille.denis@algolia.com
  display_name: Marie-Auxille Denis
  first_name: Marie-Auxille
  last_name: Denis
---

Algolia Speeds Up Search Result Delivery Times From 10 Seconds To 250
Milliseconds.The following post is a guest post by [Teddy
Chan][1], Founder and CEO at
[AfterShip][2].

![Teddy Chan AfterShip][3]

AfterShip is an online tracking platform which helps online merchants track
their shipment across multiple carriers and notify their customers via email
or mobile. Being an online merchant myself, I shipped more than 30,000
packages a month around the world. When customers contacted me to get an
update on shipments I realized that I couldn't track shipments from different
carriers and get updates on their status in a single place. So I built
Aftership to allow both consumers and online merchants view all their packages
on a single platform.

After winning the 2011 Global Startup Battle and 2011 Startup Weekend Hong
Kong Aftership opened into beta and quickly helped thousands of online
merchants to send out over 1,000,000 notifications to customers.

One of the key parts of our service is providing customers around the world
with up-to-date information about their packages.

Right now we have more than 10 million tracking numbers in our database. This
causes a few different challenges when it comes to search and we needed
technology that would help us continuously index constantly changing
information.

### Our first challenge is that we are a small team with only 1 engineer.

We are not in the search business, so we needed a solution that would be easy
to implement and work well with our existing infrastructure. Algolia's
extensive documentation made it easy to see that our set up and implementation
time would be extremely fast and would work with any language and database, so
we could get back to our core business.

**Algolia was super easy, we had it tested, up and running in a week.**

### Our second challenge was quickly delivering search results.

On Redis, searching for packages was simply impossible. For each query, it
would simply lock up until the result was found, so it could run only one
search at a time. Each search with Redis was taking up to 10 seconds. **With
Algolia we reduced search result delivery times to 250 milliseconds for any
customer anywhere in the world.** When you think about thousands of merchants
who send more than 1 million packages per month, you can see how speed is
critical.

Downtime also is not an option when tracking packages around the globe.

We are very strict when adopting new technologies and SaaS technologies can't
slow down our system.

**Algolia had the highest uptime of the other solutions we looked at. There was no physical downtime.**

### Our final challenge was search complexity.

Sometimes you need to know how many shipments are coming from Hong Kong and
exactly where they are in transit to and from the U.S.. Shipments going around
the globe can change status several times within a single day. With Algolia's
indexing we are able to instantly deliver up-to-date notifications on all 10
million packages, so that customers can not only track their package on its
journey, but they can also go to their online merchant's shop and see a real-
time status of their package.

**In the end, it was Algolia's customer service that won us over.**  
Similar services and platforms were not responsive. With Algolia we either had
the documentation we needed, immediately were able to get advice from an
engineer or had our problem solved in less than a day. With such a small team
this means a lot. And with the Enterprise package we know that Algolia will
grow with us as quickly as our business does.

**Want to find out more about the Algolia experience ?  
[ Discover and try it here ][4]**


[1]: http://hk.linkedin.com/in/teddychan/
[2]: https://www.aftership.com/
[3]: /algoliasearch-jekyll-hyde/assets/teddy.png
[4]: https://www.algolia.com/features
