---
layout: post
title: 'Concertwith.me''s Competitive Edge: A Revamped Search UX with Algolia'
author:
  login: kevin
  email: shipowlata@gmail.com
  display_name: kevin
  first_name: Kevin
  last_name: Granger
---

There are a lot of music discovery apps on the market, yet sifting through
concert listings is anything but seamless. That's why Cyprus-based startup
[Concertwith.me][1] aims to make finding local concerts
and festivals as intuitive as possible. Automatically showing upcoming events
in your area, the site offers personalized recommendations based on your
preferences and your Facebook friends' favorited music. Covering over 220,000
events globally, the site uses Algolia to offer meaningful results for
visitors who are also looking for something different.

Founder Vit Myshlaev admits that concert sites often share the same pool of
information. The differentiator is how that information is presented. "The
biggest advantage one can have is user experience," he explains. "There's
information out there, but do users find it? The reason that people don't go
to cool concerts is that they still don't know about them!"

As an example, he showed me one of the largest live music discovery sites on
the web. Searching for an artist required navigating a convoluted maze of
links before pulling up irrelevant results. "Users have to type in queries
without autocomplete, typo-tolerance, or internationalization. They have to
scroll through a long list of answers and click on paginated links. That's not
what people want in 2014," said Myshlaev.

To simplify search and make the results more relevant, Concertwith.me used our
API. "We got a lot of user feedback for natural search," Myshlaev wrote. Now
visitors can search for artists and concerts instantly. With large user bases
in the United States, Germany, France, Spain, Italy, Russia and Poland,
Concertwith.me also benefits from Algolia's [multi-lingual search
feature][2]. "We've localized our app
to many countries. For example, you can search in Russian or for artists that
are Russian, and results will still come up," says Myshlaev.

[![search][3]](https://blog.algolia.com/wp-
content/uploads/2014/08/search.gif)

For users with a less targeted idea of what they're looking for,
Concertwith.me implemented structured search via
[faceting][4]. "We also realized
that some visitors don't know what they want. Algolia search helps them find
answers to questions like, Where will my favorite artist perform? How much do
tickets cost? Are there any upcoming shows?"

[![recommendations][5]](https://blog.algolia.com/wp-
content/uploads/2014/08/recommendations.gif)

Concertwith.me's goal is to reduce informational noise so that users can find
and discover music as soon as possible. The start up experimented with a
number of other search technologies before reading an [article about us on
Intercom.io][6], which
inspired Myshlaev. "When I saw what Algolia could do, I knew that this was the
competitive edge I was looking for."

**_Want to build a search bar with multi-category auto-completion like Concertwith.me? [Learn how through our tutorial][7]._**


[1]: http://concertwith.me/
[2]: https://www.algolia.com/doc#Multilingual
[3]: /algoliasearch-jekyll-hyde/assets/search.gif
[4]: http://faq.algolia.com/basics/what-is-faceting/
[5]: /algoliasearch-jekyll-hyde/assets/recommendations.gif
[6]: http://insideintercom.io/7-things-wish-every-search-did/
[7]: https://www.algolia.com/doc/tutorials
