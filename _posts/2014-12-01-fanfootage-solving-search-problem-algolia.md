---
layout: post
title: 'FanFootage: Solving the Search problem with Algolia'
author:
  login: marie-auxille
  email: marie-auxille.denis@algolia.com
  display_name: Marie-Auxille Denis
  first_name: Marie-Auxille
  last_name: Denis
---

**The following post is a guest post by Eoin O'Driscoll (web developer), and Vinny Glennon (co-founder) of FanFootage.com**

![vinny][1]

![eoin-o-driscoll][2]

When we founded FanFootage we knew there was something lacking in the concert
and event experience. Fans were taking videos on their mobile phones during
performances, loading them on YouTube, and the audio was horrible. So we
created FanFootage where fans can upload their own video and we work with the
bands to get the high def audio and put them together. Now fans can come to
our site and see their favorite concerts or sports from any angle. They can
also search for upcoming events and performances.

For a recent Linkin Park concert the band and the fan group contacted us.
Their fans uploaded more than 1,500 videos from almost every angle. On
FanFootage that single concert has had over 350,000 page views.

![fanfootage][3]

## Our user experience heavily relies on search

Earlier in our design **we knew that search would be key to making FanFootage
the ultimate fan experience**. **When a user comes to our site the first thing
they do is search for the event or artist. And we need to make sure that they
either find the artist they are looking for or something similar, and it has
to be fast**.

As developers, our team isn't new to search, particularly within the
entertainment space. Our previous startup in the music space was bought by
RealNetworks and a second startup was a competitor to Google. That is where we
learned that search is hard. And when we thought to build our own search on
FanFootage we quickly said it wasn't going to happen.

We also know what fans need. **User demands have changed now that they can
access anything from their phones. Today we expect our applications and
services to predict what we are going to do next. And because of Google,
people don't search with a single phrase. Users expect search to understand
how phrases fit together and are related and of course it needs to spell check
and it must be instant**.

We also had different search requirements than other sites. Normally search on
a site is for one unit or concept; a site for flowers for example. For us we
needed to allow fans to search for artists, bands, friends or upcoming events
in their area and never get a zero result.

## Why we chose Algolia

After looking at a few search applications we agreed on Algolia.**Many search
applications look nice but don't have the flexibility we needed** to configure
them they way our business needed. And most weren't fast.

Why did we chose Algolia? First it has **a developer-centric approach**. It
took us 2 hours to configure and a day to test and that was it. We basically
had search up and running in a day. The dashboard lets me know that the API
calls are returned within milliseconds and we have all the flexibility we need
to configure as our content grows.

Today, more than 250 artists have used FanFootage in 20 countries. We are
growing quickly. As a company we are still learning what our fans are
searching for and Algolia is helping us with that. **As content grows we will
continue to configure search to meet the needs of our fans**. We will also be
rolling out Algolia for mobile because of its multi-search capabilities.

Algolia is a simple solution to a complex problem. And it blew our mind away.
It just works. And now we can focus on our own fanbase.

_Images courtesy of FanFootage. Learn more on their
[website][4]_


[1]: /algoliasearch-jekyll-hyde/assets/vinny.png
[2]: /algoliasearch-jekyll-hyde/assets/eoin-o-driscoll.png
[3]: /algoliasearch-jekyll-hyde/assets/fanfootage.gif
[4]: https://fanfootage.com/
