---
layout: post
title: Instant Search on CrunchBase
author:
  login: nicolas
  email: nicolas@algolia.com
  display_name: nicolas
  first_name: Nicolas
  last_name: Dessaigne
---

After launching the Beta Cloud version of Algolia, we wanted to demonstrate
what it can do. We built a search engine using CrunchBase data, so
entrepreneurs can easily search for their company or themselves. [Check it
out!][1]

You can search for companies, people and financial organizations, using
multiple attributes. Results are updated after each keystroke and matching
characters are highlighted. And of course it tolerates typos. In this post
we'll explain in more detail how it works.

## Indexation

The CrunchBase API is unfortunately rather poor. There is no way to know the
latest update. So we dump it regularly and push it to our servers in JSON
format after pruning unnecessary attributes and adding images encoded in
base64.

We actually create 3 indexes for companies (117k+ entries), persons (152k+
entries) and financial organizations (9k+ entries). The JSON files, including
images, are respectively 150MB, 70MB & 7MB.  The full indexation takes about 5
seconds (excluding upload time). Resulting index sizes are respectfully 124MB,
85MB and 7,5MB.

Indexation is done simultaneously in 3 datacenters: US-West, US-East & Europe.
Additional datacenters are on the roadmap.

## Instant Search

We trigger a query directly after page load and again after each keystroke. To
simplify communication with the server, we created a javascript client
(contact us if you want to use it before its release). We then simply call the
search function indicating the callback that will handle resulting hits
asynchronously. More details to follow once we've written the doc!

We automatically choose the server closest to your location by using [Amazon
Route 53][2]. Once the DNS lookup is resolved, it
lets us get low enough latencies that the response feels nearly instantaneous
(if you test it from North America or Europe). From DSL connections, we obtain
search latencies of about 90ms in San Francisco, 75ms in New York and 65ms in
London. About 20ms are used for querying the index, 5ms for compressing the
data and 5ms for uncompressing. The remaining time is the actual transfer of
the data and depends of your location and the quality of your connection.

If you're a hacker, you may also remark the presence of an API key in the
javascript. It cannot be hidden as we directly query our servers from the
browser. The operations it enables are however restricted to search only, you
would need to use a different key to update entries for example. You can
create and revoke as many API keys you need directly from the API.

## Hits Display

No designer worked on the demo, but we hope it doesn't show! We execute the 3
queries simultaneously and display the results by blocks of 20 hits.
Additional queries are automatically triggered when scrolling to the bottom of
the page.

We display approximate results with a transparent background to clearly
differentiate them.

You can use the arrow keys to navigate inside the results.

## Ranking

We use the standard ranking order. By descending priority:

  * Exact matches before approximate matches;
  * User-defined order of attributes;
  * Distance between the matching term and the beginning of the attribute;
  * Proximity between terms in multi-word queries;
  * User defined score.

For the order of attributes, we use {name, twitter, organization or people,
description}. This translates into very simple settings. Here are the settings
of the persons index, for example:

    
    {
      "attributesToIndex": ["name", "twitter", "unordered(companies)", "description"],
      "attributesToHighlight": ["name", "twitter", "companies", "description"],
      "customRanking": ["desc(size)", "asc(name)"]
    }

By default, all attributes are indexed and highlighted: "attributesToIndex" &
"attributesToHighlight" enable us to precisely define what to index (and in
what order) and what to highlight. The "unordered" modifier disable ranking
between values of a multi-valued attributes.

For the user defined score ("customRanking" in settings) , we sort by
decreasing order of CrunchBase entry size and then by alphabetical order.

## Help us

This is just a demo but we'd like to continue improving it! Please tell us
what you think and send your suggestions: contact at algolia dot com


[1]: http://www.algolia.com/demo/crunchbase/
[2]: http://aws.amazon.com/route53/
