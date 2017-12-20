---
layout: post
title: Algolia Search Beta 2 is out!
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

We are pleased to announce the launch of Algolia Search Beta 2, our second
release!

We would like to sincerely thank all of our beta testers for the great
feedback. You really helped us to make Algolia Search a first-class product
guys! Please continue your feedbacks!

And here what's new in this beta2.

**iOS & Android changes:**

  * Improved performance for big data sets (up to three times faster). In our tests, we successfully used a 3 millions entries data set on a old iPhone 3GS. The index was 250MB large!

**Documentation changes:**

  * Reworked the overview
  * Fixed a lot of typos and small errors

**iOS specific changes:**

  * Added an AlgoliaSearch.h header that includes all public headers
  * Prefixed all public classes by AS
  * Changed addEntry selector in ASIndexWriter to be more similar to NSMutableDictionary API
  * Removed internal objects from public headers
  * Changed ASAsyncIndexSearcher API to implement the delegate pattern


