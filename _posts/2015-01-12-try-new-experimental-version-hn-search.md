---
layout: post
title: New experimental version of Hacker News Search built with Algolia
author:
  login: kevin
  email: shipowlata@gmail.com
  display_name: kevin
  first_name: Kevin
  last_name: Granger
---

Exactly a year ago, we began to power the Hacker News search engine (see our
[blog post][1]. Since then,
our HN search project has grown a lot, expanding **from 20M to 25M indexed
items**, and serving** from 900K to 30M searches a month**.

In addition to [hn.algolia.com][2] we're also providing

build various readers or monitor tools and we love the applications you're
building on top of us. The community was also pretty active on
[GitHub][4], requesting improvements
and catching bugs... keep on contributing!

## Eating our own dog food on HN search

We are **power users of Hacker News** and there isn't a single day we don't
use it. Being able to use our own engine on a tool that is so important to us
has been a unique opportunity to **eat our ****own dog food.** We've added a
lot of API features during the year but unfortunately didn't have the time to
refresh the UI so far.

One of our 2015 resolutions was to push the envelope of the HN search UI/UX:

  * make it more **readable**,
  * more **usable**,
  * and use **modern** frontend frameworks.

That's what motivated us to release a [new experimental version of HN
Search][5]. Try it out and tell us what
you think!

## Applying more UI best practices

We've learned a lot of things from the
[comments][6] of the users of the
previous version. We also took a look at all the [cool
apps][7] built on top of our API. We wanted to
apply more UI best practices and here is what we ended with:

![][8]

### Focus on instantaneity

The whole layout has been designed to provide an instant experience, reducing
the wait time before the actual content is displayed. It's also a way to
reduce the number of mouse clicks needed to access and navigate through the
content. The danger with that kind of structure can be to end up with a
flickering UI where each user action redraw the page, activating unwanted
behaviors and consuming a huge amount of memory.We focused on a smooth
experience. Some of the techniques used are based on basic performance
optimizations but in the end what really matters for us is the user's
perception of latency between each interactions, more than objective
performance. Here are some of the tricks we applied:

  * **Toggle comments**: we wanted the user to be able to read all the comments of a story on the same page, our API on top of Firebase allowed us to load and display them with a single call.
  * **Sticky posts**: in some cases we are loading up to 500 comments, we wanted the user to be able to keep the information of what he is reading and easily collapse it, so we decided to keep the initial post on top of the list.
  * **Lazy-loading of non-cached images**: when you are refreshing the UI for each request you don't want every thumbnail to flick on the UI when loading. So we applied a simple fade to avoid that. But there is actually no way to know if an image is already loaded or not from a previous query. We manage to detect that with a small timeout.
  * **Loading feedback**: the most important part of a reactive UI is to always give the user a feedback on the state of the UI. We choose to add this information with a thin loading bar on top of the page.
  * **Deferring the load of some unnecessary elements**: this one is about performance. When you are displaying about 20 repeatable items on each keypress you want them as light as possible. In our case we are using Angular.js with some directives which were too slow to render. So we ended up rendering them only if the user interact with them.
  * **Cache every requests**: It's mainly about the backspace key. When a user want to modify his query by removing some characters you don't want to make him wait for the result: that's cached by the Algolia JS API client.

### Focus on readability

We've learned a lot from your comments while releasing our first HN Search
version last year. Readability of the search results must be outstanding to
allow you to quickly understand why the results are retrieved and what they
are about. We ended up with 2 gray colors and 2 font weights to ease the
readability without being too distracting.

![][9]

### Stay as minimal as possible

If you see unnecessary stuffs, please tell us. We are not looking for the most
'minimal' UI but for the right balance between usability and minimalism.

### Sorting & Filtering improvements

Most HN Search users are advanced users. They know exactly what they are
searching for and want to have the ability to sort and filter their results
precisely. We are now exposing a simple way to either sort results by date or
popularity in addition to the period filtering capabilities we already had.

### Inlined comments

We thought it could make a lot of sense to be able to read the comments of a
story directly from the search result page. Keeping in mind it should be super
readable, we went for indentations & author colored avatars making it really
clear to understand who is replying.

![][10]

### Search settings

Because HN Search users are advanced users, they want to be able to customize
the way the default ranking is working. So be it, we've just exposed a subset
of the underlying settings we're using for the search to let you customize it.

![][11]

### Front page

Since Firebase is providing the official API of Hacker News, fetching the
items currently displayed on the front page is really easy. We decided to pair
it with our search, allowing users to search for hot stories & comments
through a discreet menu item.

![][12]

### Starred

Let's go further; what about being able to star some stories to be able to
search in them later? You're now able to star any stories directly from the
results page. The stars are stored locally in your browser for now. Let us
know if you find the feature valuable!

![][13]

## Contribution

As you may know, the whole source code of the HN Search website is open-source
and hosted on GitHub. This new version is still based on a Rails 4 project and
uses Angular.js as the frontend framework. We've improved the README to help
you being able to contribute in minutes. Not to mention: we love pull-
requests.

**Now is starting again the most important part of this project, user testing**. We count on you to bring us the necessary information to make this search your favorite one.

## Wanna test?

To try it, go [to our experimental version of HN Search][14], go to "Settings", and enable the new style:

![][15]

![][16]

### Want to contribute?

It's open-source and we'll be happy to get your feedback! Just use [GitHub's
issues][17] to report any idea you
have in mind. We also love pull-requests :)

Source code: [https://github.com/algolia/hn-search](https://github.com/algolia
/hn-search)

## [Try it now!][18]


[1]: http://blog.algolia.com/hacker-news-search-algolia/)
[2]: http://hn.algolia.com
[3]: http://hn.algolia.com/api
[4]: https://github.com/algolia/hn-search/issues
[5]: https://new-hn.algolia.com/?experimental
[6]: https://news.ycombinator.com/item?id=7451206
[7]: http://hn.algolia.com/cool_apps
[8]: /algoliasearch-jekyll-hyde/assets/hackpad.com_JoORx6jqcVU_p.233467_1420906940736_Screen%20Shot%202015-01-10%20at%2017.22.06.png
[9]:  /algoliasearch-jekyll-hyde/assets/hackpad.com_JoORx6jqcVU_p.233467_1420909252393_Screen%20Shot%202015-01-10%20at%2018.00.12.png
[10]: /algoliasearch-jekyll-hyde/assets/hackpad.com_JoORx6jqcVU_p.233467_1420907561110_Screen%20Shot%202015-01-10%20at%2017.32.21.png
[11]: /algoliasearch-jekyll-hyde/assets/hackpad.com_JoORx6jqcVU_p.233467_1420907721098_Screen%20Shot%202015-01-10%20at%2017.35.07.png
[12]: /algoliasearch-jekyll-hyde/assets/hackpad.com_JoORx6jqcVU_p.233467_1420907937295_Screen%20Shot%202015-01-10%20at%2017.38.44.png
[13]: /algoliasearch-jekyll-hyde/assets/hackpad.com_JoORx6jqcVU_p.233467_1420908337814_Screen%20Shot%202015-01-10%20at%2017.45.01.png
[14]: https://new-hn.algolia.com
[15]: /algoliasearch-jekyll-hyde/assets/hackpad.com_JoORx6jqcVU_p.233467_1420907058703_Screen%20Shot%202015-01-10%20at%2017.23.21.png
[16]: /algoliasearch-jekyll-hyde/assets/hackpad.com_JoORx6jqcVU_p.233467_1420907103965_Screen%20Shot%202015-01-10%20at%2017.24.51.png
[17]: https://github.com/algolia/hn-search/issues
[18]: https://new-hn.algolia.com/?experimental
