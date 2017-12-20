---
layout: post
title: Painless integration, crystal clear documentation, please welcome Algolia Search
  beta 4!
author:
  login: nicolas
  email: nicolas@algolia.com
  display_name: nicolas
  first_name: Nicolas
  last_name: Dessaigne
---

[![][1]](https://blog.algolia.com
/wp-content/uploads/2012/10/CitiesSuggestIPhone5Slide3.png)

Last month has been truly electrifying! We joined our friends at
[Yakaz][2] in their office space, we participated in many
events... and most exciting of all, we spent days and nights reworking the
product! Today we are really proud to present you the result of this time well
spent!

I know I told some of you beta3 would be the last, but we could not ignore
your excellent feedback. So here comes Algolia Search beta4, a true revolution
(I hope this is not trademarked!) in mobile search!

Here come the major improvements:

  * A completely **reworked API** that we just love to use everywhere. The time to fully understand the library has been reduced to nearly nothing thanks to all your feedback. We are proud to have the easiest to use search library ever made!
  * A completely **rewritten documentation** with detailed API and step-by-step tutorials. You should be able to make your first queries in a matter of minutes!
  * **Easy highlight** for **multiple fields queries**. A bit cryptic? Stay with me... just imagine a Contact application where you can search for your contact by any field, i.e., name, company, address or even notes. Algolia Search now provides easy-to-use highlighting of any matching words. It is even able to generate a snippet when the text is too long. Check out the tutorials to learn more! Highlighting relevant results just became child's play!
  * A greatly **improved out-of-the-box relevance**. Our mission is to simplify search: we want the best possible relevance by default so you can forget these long hours of tuning :)

But that's not all, many smaller improvements were also included in this
release:

  * Support of **advanced queries**. Take again our tutorial contact application, wouldn't it be great to be able to search by initials? You can now implement this cool feature in a couple of minutes without any headache on relevance tuning.
  * Support for **ARC and no ARC**. In the previous beta we added an iOS version for people that do not use [Automatic Reference Counting (ARC)][3]. We now have only one version that supports projects with and without ARC. If you do not use ARC, all objects received from Algolia Search are autoreleased.
  * Support for **user objects backward compatibility**. As the index is also an objects store, you can modify your object members and still read older indexes! It is very easy to implement, just check the tutorials.
  * The release also fixes a few bugs that we discovered during your and our intensive testing.

It would not have been possible without the help of our many beta testers,
thank you all! Special thanks to [Kris][4],
[Hoa][5] and
[Thomas][6] whose guidance has been priceless.

So... what's next? Many things! This time I really believe this is the last
beta. The final release is just around the corner. Of course, we appreciate
your feedback nonetheless and always will! You can also expect a new website
and a few apps in the app store! Who said a contact app?

Stay tuned!


[1]: /algoliasearch-jekyll-hyde/assets/CitiesSuggestIPhone5Slide3-300x194.png
[2]: http://www.yakaz.com/
[3]: http://developer.apple.com/library/mac/#releasenotes/ObjectiveC/RN-TransitioningToARC/Introduction/Introduction.html
[4]: https://twitter.com/krmarkel
[5]: https://twitter.com/dinh_viet_hoa
[6]: https://twitter.com/sarfata
