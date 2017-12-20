---
layout: post
title: Algolia Search Offline 2.1 is out!
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

We are pleased to introduce a new version of Algolia Search Offline for iOS,
Android and OS X (Windows versions will come soon).

Version 2.1 significantly improves the out-of-the box relevance of Algolia
Search. We are now confident we have the best relevance on the market. We will
discuss our ranking approach compared to traditional methods in another post.
For now, the two main improvements offered in this version are:

    * When the query string has one word that is the entire content of a field, we will display it first. You can try it with the "arm" query on our Crunchbase demonstration to get the idea: [http://bit.ly/searchcrunchbase](http://bit.ly/searchcrunchbase) (it uses the same relevance approach).
  * More importance is given to proximity than to the order of attributes. For example, the query "managing p" will now match first "Managing Partner" in the "Title" attribute instead of "P" in the "Name" attribute followed by "Managing" in the "Title". This is the case even if the order of the attributes is ["Name", "Title", ...].

While you can control ranking with the setRankingOrder method, you will
benefit from these improvements by default.

This new release also introduces some new features:

  * A way to efficiently serialize latitude/longitude and float values in your custom objects (reduce the size of serialized objects by up to 80%).
  * A method to compile an index in an old version format. This is useful when indexes are created server side and then pushed to applications that support old versions of Algolia.
  * All characters in tag filters are now supported.
  * It is now possible to do a logic OR between tags in filters. For example, you can search all contacts that match the query "paul" and have the tag "friend" OR the tag "classmate".

We hope you'll like these new features, and as ever, we welcome your feedback!


