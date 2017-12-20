---
layout: post
title: Algolia Search Beta 3 is out!
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

[![Algolia Search Beta 3][1]](https://blog.algolia.com/wp-
content/uploads/2012/09/CitiesSuggestIOSPortraitSmall.png)We are pleased to
announce Algolia Search Beta 3, our third release with a strong focus on
performance.

As for the previous release, we would like to sincerely thank all of our beta
testers for their excellent feedbacks!

Here what's new in this beta3.

  
**iOS & Android changes:**

  * **Ultra fast loading**: indexes are now loaded in a few milliseconds (always less  
than 10ms!)  With the Beta 2, an index of 500MB could take up to 20 seconds to
load.

  * **Ultra fast search on big indexes**: Beta 2 was able to search up to 100k entries  
in real time. Beta3 can search in **5M entries in real time** (and probably

more!). Our main use case was to search in all titles of the English version
of

Wikipedia on a IPhone 3GS. The speedup is also very nice for small datasets,

the near-zero CPU usage increases battery life compared to Beta 2.

  * Highlight is now always done on longest match. In previous version a query 'anq' could highlight "Angeles" in  
two different ways: "**Ang**eles" or "**An**geles", with this version you will
always obtain "**Ang**eles" which is easier to

understand for end-users.

  * Improved proximity scoring when a query contains multiple words.
  * Fixed two memory leaks that could lead to problems with very heavy usage.

**iOS specific changes:**

  * Added a version without ARC that allows to target iOS >= 3.0


[1]: /algoliasearch-jekyll-hyde/assets/CitiesSuggestIOSPortraitSmall-155x300.png
