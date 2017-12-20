---
layout: post
title: Great discussions at LeWeb'12 London
author:
  login: nicolas
  email: nicolas@algolia.com
  display_name: nicolas
  first_name: Nicolas
  last_name: Dessaigne
---

On June 19 & 20th, I had the chance to participate to [LeWeb 2012
London][1] edition. This year theme was "Faster than
Real Time" and we had an impressive list of speakers! But the true value of
LeWeb is elsewhere: It's in the 1283 people from 52 countries who were present
and with whom you could network!

They chose [Presdo Match][2] to help people meet and
honestly... this tool would benefit from some improvements, especially a
mobile version! Still, I was able to find no fewer than 100 participants
having the "mobile" keyword in their profile and, from there, organize a
handful of meetings. Thanks to all of you who accepted to meet me or whom I
met unplanned, with a special thanks to [Paul Ardeleanu][3],
[Gora Sudindranath][4], [Lindsey C.
Holmes][5], [Marius
Rostad][6], [Kevin
McDonagh][7] and [Alexandre
Delivet][8] for their precious feedbacks about
Algolia.

[caption id="attachment_82" align="alignright" width="180"][![Cities Suggest
Demo][9]](https://blog.algolia.com/wp-
content/uploads/2012/07/citiessuggest-leweb.png) Cities Suggest demo @
LeWeb[/caption]

I had the opportunity to do the very first demonstrations of our instant
suggest lib, and that was both exhilarating and frustrating! We chose to
develop a small proof of concept suggesting city names from anywhere in the
world. Here's what I learned:

  * A demo is better than many words! Even if most people knew what I meant by "google instant suggest", the demo was key in clarifying our offering.
  * Even if we chose cities because it was easy to demonstrate (thanks to the geonames database), it can be interesting in itself!
  * 100ms seemed a pretty fast response time in our initial testing, but it's actually way too slow to have a smooth user experience.

Over all that was a very good experience, and I came back with a few
improvements to implement (most coming from my own frustration showing the
demo while the feedback was actually very positive!)

The most important piece of feedback was about the perceived sluggishness of
the app. We decided to implement an asynchronous version of the lib. Beware,
it actually comes with a drawback for our users; It's significantly more
difficult to integrate. But it did not take long for us to decide it was the
way to go, since the perception of speed is so natural that the benefit far
outweighs the longer integration code. We'll now work on simplifying it!

We'll soon do a post about this demo. In the meantime, stay tuned!


[1]: http://london.leweb.co/
[2]: http://match.presdo.com/
[3]: http://hello24.com/
[4]: http://www.linkedin.com/in/goras
[5]: http://twitter.com/lindseycholmes
[6]: https://twitter.com/#!/portart
[7]: https://twitter.com/#!/kevinmcdonagh
[8]: http://www.alexdelivet.com/
[9]: /algoliasearch-jekyll-hyde/assets/citiessuggest-leweb-180x300.png
