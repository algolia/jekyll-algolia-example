---
layout: post
title: C/C++ is still the only way to have high performance on Mobile
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

When it comes to programming languages and performance, you can read all and
its opposite on the web. It's definitely a very controversial topic!

**[Edit 15-Nov-2012]** I had questions on [reddit][1] about the data-structures and algorithms we used. We develop an [embedded search engine][2] for mobile, and tests were done on our own data-structure that is far more efficient than SQLite or other platform options for this use-case. **[/Edit]**

For Algolia the story started when researching an instant suggest algorithm. I
used Java for two reasons:

  * **Main reason**: our first client was using Java on Google App Engine
  * **Secondary**: at that stage, I was doing a lots of refactoring and Eclipse is very efficient for these tasks

Once our algorithm was designed, I started to optimize performance on a
desktop computer (core I7 950). For this, I indexed all titles of the English
version of Wikipedia (4 millions titles). I optimized the Java code mainly by
reducing the number of allocations. All instant suggest queries were then
faster than 10ms.

  
As we planned from the begining to support iOS and Android, I needed to
optimize for high performance on mobile. I then ported the Java code to
Android and ended up with a few modifications (we needed to support old
Android versions which have not a full support of Java SDK).

In order to test performance, I created a small "demo" app: CitiesSuggest, a
mobile application based on Geonames database to look for a city name. I
filled the index with all places with a known population. In the end, the
database contained 270 000 city names.

As could be expected, the resulting application was quite sluggish on my
Galaxy S2. The worst queries could take more than one second of CPU.

I then applied all possible methods described in the [Anroid
documentation][3]
and was able to reduce the response time to 700ms for the longest queries.
This is better but still gives end-users an impression of sluggishness!
Fortunately, common queries worked well enough to present our very first demos
at [LeWeb'12 London][4]. I was subsequently able to improve the user experience a lot
by adding asynchronous support. At that point, we decided to start
implementing the objective-C version for iOS.

I started a new implementation fully done in objective-C from scratch without
any premature optimization. I then developed the same CitiesSuggest
application for iOS. I first got stuck with some basic UI stuff. For example I
needed to reimplement an UILabel that supports highlighting! The standard
UILabel does no support this, and other implementations like Three20 just have
too many dependencies (you can download AlgoliaUILabel on
[github][5], I released  the code under Apache
licence). Once the UI was ready, I could see the actual performance was
catastrophic! Instant suggest queries were between 200ms and 10 seconds on an
iPhone 4S!

After profilling, I discovered that 95% of the time was spent in Objective-C
messaging and ARC. Actually, I have millions of calls to very small methods
with 1 or 2 lines of code, and I found a very good explanation in the internal
of objc_msgSend (mainly on [mikeask.com][6]. There is in fact a hash table
lookup behind objc_msgSend! This explains most of my performance problems.

To fix these problems, I started to replace all this low level objective-C
implementation by a C++ implementation with inlined methods. Eventually, I
finished with Objective-C code for high level classes while all the core was
C++.

This change has dramatically improved performance with instant-search queries
between 2ms and 90ms on a iPhone 4S. I struggled to find complex enough
queries to reach 90ms! This resulted in a very nice user experience with a
remarkably slick demo :)

After this succes, I decided to use the same C++ code on Android with Android
NDK. With this approach I reduced our maximum query time from 700ms to 80ms on
a Galaxy S2. But I was really disappointed by the Android display, as I did
not reach the same level of interactive experience that I had with iOS. The
display of results stays slower on Android, probably because I did not spend
enough time to optimize this part.

In the end, I lost a lot of time with Java and Objective-C trying to optimize
code when the real solution was to use C/C++. I am fully convinced that it is
just not possible to reach the same speed with Objective-C only or Java only.

And there is also another good property with C++ code: Blackberry supports
C++, and there is large chance that Windows Phone 8 SDK will support C++ when
released in a few weeks. Actually, I do not see any other alternative than
C/C++ when you are looking for performance on mobile devices, which is our
case :)

I hope this article will prevent you from spending as much time on
Java/Objective-C optimizations as I did.


[1]: http://redd.it/136hny
[2]: http://www.algolia.com
[3]: http://developer.android.com/guide/practices/performance.html
[4]: http://blog.algolia.com/great-discussions-at-leweb12-london/
[5]: http://github.com/algolia/UILabel
[6]: http://www.mikeash.com/pyblog/friday-qa-2009-03-20-objective-c-messaging.html)
