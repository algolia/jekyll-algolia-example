---
layout: post
title: 'V2: Search by Geolocation in our Offline Search SDK'
author:
  login: nicolas
  email: nicolas@algolia.com
  display_name: nicolas
  first_name: Nicolas
  last_name: Dessaigne
---

While our latest news focused on the Algolia Search cloud offer (you can still
join the [beta][1], we're
pleased to introduce a major new version of Algolia Search offline: V2! It is
available today for iOS, Android and OS X. Windows Phone and Windows versions
will be released as soon as they are ready. A few months in the making, these
new features were built on early customer feedback and will simplify
integration.

## Algolia becomes the easiest way to search by geolocation!

The ease of integration is a constant concern for us and that's why we
carefully consider every new feature. Two important features made it in this
version:

  * Geo-search means the ability to search around a location or inside a bounding box. Results can be sorted by distance and of course geo-queries can be combined with textual ones. We added a dedicated tutorial in the doc to get up to speed with this new feature in no time (for [iOS][2] and [Android][3].
  * Tag filters enable restriction of results to specific tags. We received this demand a number of times in order to avoid the creation of too many specialized indexes.

These new features are also available in the beta of our cloud version!

## Improved performance and ranking

With some hard work... and a lot of profiling, we have been able to get a 10%
gain in performance on every query.

In V1, name matches were always considered more important than other
attributes, but we didn't consider differences between other attributes. This
changed in V2: ranking priority now respects the order in which you indicate
attributes in the textToIndex method. It's more powerful while actually being
more consistent with no specific processing of the name field.

But this improvement comes at the cost of a slightly bigger index and longer
computation. If index size is important or if you need to earn a few
nanoseconds more, you can optimize it away with the increaseCompression
method. You'll get a 10 to 30% reduction in index size and an additional 20%
boost in performance (that's 30% total compared to V1!).

## Easy just got easier

Integrating search in an app has never been so easy. For V2 we took into
account all the excellent feedback we received, and wherever it was possible
we simplified the API:

  * No distinction between suggest and search methods. We wanted to match the expected use-cases of the SDK but it was causing more confusion than anything else. So there is now only one way to send queries to an index: the search method.
  * With the addition of geo-search, the index class was becoming crowded. We simplified this by decoupling the search approach and query definition. A small set of search methods enable the developer to choose if the search will be synchronous, use a callback, or batch several queries. And a simple SearchQuery class defines the nature of the queries themselves: geolocation, use of prefixes, tag filters, etc.
  * Out of simple strings for which we provide a helper, every indexable object now has a UID. Our use of a "name" for this role led to a few difficulties when collisions were possible (persons for example). There are no longer any privileged attributes.
  * License key initialization is now done using a static method. It is a best practice that was actually necessary to build a [RubyMotion gem][4].

Specific to Android, we also added an AbstractIndexable abstract class.
Instead of implementing the Indexable interface, you now have the option of
directly extending AbstractIndexable that takes care of optional methods for
you.

Specific to iOS, you can now directly index core data entities with the
setCoreDataEntityDescription selector. No need to create a wrapper.

## Still able to read V1 indexes

If for any reason you cannot replace or reindex your data, V2 is still able to
search in a V1 index. However, as the name attribute was removed you do need
to implement the IndexableLegacy interface. If you then publish changes, the
new index will be in the V2 format.

We're really sorry to make our Windows Phone and Windows customers wait. Feel
free to torment us with your needs, it's great motivation to finish more
quickly ;)

If you're still reading, I guess it's time for you to test this new version of
the Algolia Search Offline SDK. [Get started][5]!


[1]: http://blog.algolia.com/our-saas-version-is-in-beta/)
[2]: http://www.algolia.com/doc/ios/#iOS_Geoloc
[3]: http://www.algolia.com/doc/android/#android_Geoloc)
[4]: http://blog.algolia.com/algolia-search-is-now-available-for-rubymotion/
[5]: http://www.algolia.com/get-started/
