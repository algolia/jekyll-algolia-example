---
layout: post
title: Algolia Search is now in Beta for Microsoft Windows 8 Desktops and Tablets
  - The Algolia Blog
author:
  login: nicolas
  email: nicolas@algolia.com
  display_name: nicolas
  first_name: Nicolas
  last_name: Dessaigne
---

You may have been expecting Windows Phone 8 as our next platform of choice,
but we preferred to go Windows 8 first! With more than 40M sold in a month,
let's say it's a slightly more interesting market for now.

## Windows 8: Our First Move to the Desktop

Yes, desktop apps are not as dead as many people think, especially now that
both Mac OSX and Windows come with their app stores. Sure, Windows is now
running on tablets - and we also support them - but it will take time before
they surpass desktops.

What's more, the new Windows 8 <del>Metro</del> Modern UI includes a charm bar
with search and autocompletion as the number one feature. Microsoft actually
provides an interface to ease application integration in the charm search
bar... but no library to help you implement it. No problem, here comes Algolia
Search for Windows 8!

## Awesome Features for Your Apps

The core technology is exactly the same as the one in our iOS and Android
versions. That means you'll get exactly the same exceptional performance...
well probably even better depending of the hardware you're running on. You'll
also get instant visual feedback, typo-tolerance and multiple attributes
support! See all the features on our
[website][1].

## Easy to Integrate in your Language of Choice

Check out the dedicated [tutorials][2]! The
Algolia Search library has been designed to work directly on the platform.
That means you can use it in whatever language you prefer, be it JS or one of
the .Net choices (C#, VB, C++). The tutorials are available in both JS and in
C#.

Moreover it is fully compatible with Microsoft **Search Contracts**. It just
became child's play to integrate Search in your Windows app!

## Hey Microsoft, There are a Couple of Features You Could Improve

Windows 8 is clearly a step in the right direction, but we encountered some
problems we didn't expect!

First is the multi-arch support. While it's just straightforward with iOS or
Android (out of the JNI part...), using a native lib forces the developer to
chose an architecture on Windows! Fortunately, Visual Studio Package Generator
handles that correctly and proposes you to select all architectures for your
final export. But well... we'd have preferred it plain and simple.

Our second deception is a bit more problematic as it prevents one of our very
nice features: typo-tolerant highlighting during autocompletion. It starts
with best of intentions from Microsoft. To remove the burden of implementing
highlight for autocompletion on developers, Windows 8 handles it directly. The
only problem is that it cannot be replaced by our own :( So you can obtain
hits even if your query contains typos, but hits containing text different
from the query won't be highlighted.

We hope to be able to change this behaviour in the future. In the meantime,
we're impatient to get your feedback on this beta!
[Register][3] for your Windows 8 version of Algolia
Search now!


[1]: http://www.algolia.com/product/
[2]: http://www.algolia.com/doc/win8/
[3]: http://www.algolia.com/try/
