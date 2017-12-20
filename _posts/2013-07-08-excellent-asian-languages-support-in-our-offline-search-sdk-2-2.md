---
layout: post
title: Asian Language support in our Offline Search SDK 2.2
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

Like most search engines, version 2.1 did not include any specific processing
for Asian Languages. Version 2.2 significantly improves Asian language support
(Chinese, Japanese, Korean) by including specific processing like the
automatic conversion between Simplified Chinese and Traditional Chinese using
the [Unicode UniHan Database](http://www.unicode.org/charts/unihan.html). This
advanced processing was only possible because we built our own [Unicode
library](http://blog.algolia.com/why-develop-our-own-unicode-library/). Many
thanks to [Stephen](https://twitter.com/stephencremin) for his help!

This release also contains other improvements we released first for our SaaS
version:

  * The out-of-the-box ranking was greatly improved when queries contained more than two words,
  * Indexing speed was greatly improved on mobile (2 times more efficient),
  * Search speed was improved by about 20%.

We hope you’ll like these new features, and as ever, we welcome your feedback!

