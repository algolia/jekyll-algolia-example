---
layout: post
title: Inside GrowthHackers.com's Implementation of Algolia
author:
  login: maxime
  email: maxime.locqueville@algolia.com
  display_name: maxime
  first_name: Maxime
  last_name: Locqueville
---

We interviewed [Dylan La Com][1], Growth Product
Manager at [Qualaroo][2] &
[GrowthHackers.com][3], about their Algolia
implementation experience.

[![growthacker][4]](https://blog.algolia.com/wp-
content/uploads/2014/05/growthacker.jpg)

### What role did search play at GrowthHackers before the Algolia
implementation?

When we launched our community site
[GrowthHackers.com][5] in October 2013, search was
admittedly an afterthought for us. GrowthHackers is a social-voting site where
marketers, founders, and product-people can share and discuss growth-related
content. At launch, it was unclear what role search would have on the site.
GrowthHackers is built on Wordpress, and with that comes Wordpress' standard
search functionality. What WP search does is append an additional keyword or
phrase parameter to its typical post query and load a new page with the
results. WP search only indexed the outbound URLs of the articles our members
submitted, and this made finding specific content difficult.

### Why did you want to give search an update on GrowthHackers?

We started hearing about our lack of a solid search feature from some of our
more active users. One of our members even put together a slide presentation
to prove just how useless our search was [[check it out
here][6]]. At the same time, GrowthHackers was becoming more than just a
way to stay up-to-date on the best growth articles, it was becoming the place
to get answers: an encyclopedia for growth-related information. Search volume
at this time was peaking in the mid-hundreds per week. We needed a search
feature that could support this evolving use-case.

### Why did you choose Algolia?

We looked at several search solutions before trying Algolia, including
Swiftype, WP Search (plugin), and Srch2. All are great solutions, but
ultimately, we went with Algolia because they had the right mix of features:
Their integration was simple, the documentation was thorough, and there were
plenty of starter templates. I knew it was a good sign when, while looking
their GitHub repository, I found they had a demo site built with search that
worked very similar to how we hoped ours would work, complete with real-time
results, typo-tolerance, and filters. The Algolia team was incredibly helpful
getting us set up and was there each step of the way through the integration
process, providing resources and best practices for creating a truly top-notch
search experience.

### Tell me a little about how the new search works.

Our primary use of Algolia is to store and index user submitted content, and
provide real-time search in our growing database of growth-related articles,
questions, videos and slides. The majority of what we index is article titles
and URLs--strings which are generally small. Visitors to our site often come
with specific growth-related questions and use our search to find answers
quickly. For example, someone interested in learning best practices for
running Twitter ads could type in "Twitter ad" and within milliseconds see
dozens of articles and discussions related to maximizing ROI for Twitter ads.
Using Algolia's admin dashboard, we're able to set ranking priorities based on
the number of votes and comments of each article, and make sure the top
results are the most relevant. So, the visitor who searches "Twitter ad" is
shown articles with the highest mix of votes and comments. Algolia took the
search ranking process and wrapped it in a clean and simple interface that
allows anyone, regardless of their experience with search, to easily adjust
and manipulate.

One of the challenges we faced during the integration process was
understanding how to keep our main database synced and up to date with our
Algolia index. User submitted content on GrowthHackers changes often as users
interact with the content. Each post once submitted may receive upvotes and
comments from members in the community. Each post also has a wiki-style
summary field that can be edited by community members. Lastly, posts can have
several states, including published, pending and trashed. In order to ensure
our content on Algolia mirrored the content in our database, we set up a job
queue and a cron process to periodically push updates to our Algolia index.
This has been working quite well for us.

### How has the new search impacted engagement?

We released the new search mid-February, and since the release we've seen
search volume increase 4-5X. Of course there are several factors at play here,
including increased traffic volume and better search bar placement, but it is
clear that Algolia's search features have contributed to an impressive
increase in search engagement. On average, visitors who utilize search view
2-3X more pages per session and spend 5-6X longer on the site than those who
don't search. Algolia's analytics dashboard provides us with an incredible
glimpse of visitor intent on our site by showing us the queries visitors are
searching for, and trend lines to show popularity over time. With this data,
we're able to better understand how our visitors want to use our site, and
make better decisions about how to organize the content.

Moving forward, we're hoping to implement Algolia's search filters to provide
even better ways to access content on our site. We're excited to have such a
powerful tool in our stack and hope to experiment with new ways to provide
search functionality throughout GrowthHackers.


[1]: https://twitter.com/dylanLaCom
[2]: https://qualaroo.com
[3]: http://growthhackers.com
[4]: /algoliasearch-jekyll-hyde/assets/growthacker.jpg
[5]: http://growthhackers.com/
[6]: http://www.slideshare.net/andrewmatthewthompson/improving-search-on-growthhackers
