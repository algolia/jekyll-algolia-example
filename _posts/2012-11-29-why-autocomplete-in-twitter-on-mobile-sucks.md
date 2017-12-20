---
layout: post
title: Why Autocomplete in Twitter Mobile App Sucks
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

[![A better Twitter mobile app experience with Algolia
Search.][1]](https://blog.algolia.com/wp-
content/uploads/2012/11/ScreenshotTwitter.png)Autocomplete is so
intuitive,that it seems like it would be easy to implement. However, most
mobile apps that offer it provide a pretty poor user experience. Let's look at
the Twitter mobile app as an example.

Twitter proposes autocompletion when you create a new tweet. The idea is to
make suggestions after the '#' and '@' characters. It's actually very nice to
gain time, especially when you're tweeting with a small virtual keyboard...
but it sucks!

## Avoid Roundtrips to Server for Autocompletion

The first reason is that when you're on the go, latency is often too high on
mobile, leading to unusable autocomplete -  except if you're very slow to
type. Twitter developers chose to develop this functionality server-side,
probably with lucene, and to expose it via APIs to their mobile app. That's
good for reusability but not so much for usability...

## Beware of the Suggestions Ranking

The second reason is the ranking is just obscure. Yesterday I sent a tweet to
@cocoanetics and the screenshot on the left shows the suggestions I got when
typing "@c". I would greatly prefer to see Twitter handles before names and it
would never come to my mind to look for "Marie Cecile" with "@c"!

## Explain the Matches

Last but not least there is no visual feedback to show me why the app proposes
a given user. So ok let me think... the 'c' was reffering to "Cécile" in
"Marie-Cécile"! A bit far fetched!

Now let's imagine the Twitter mobile app with instant autocompletion even
offline, intuitive ranking, and visual feedback... Appealing, isn't it?
Twitter if you listen, check it up, I'm sure you'll love [Algolia
Search][2]!


[1]: /algoliasearch-jekyll-hyde/assets/ScreenshotTwitter.png
[2]: http://www.algolia.com/product/
