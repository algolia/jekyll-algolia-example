---
layout: post
title: Deploying Algolia to Search on more than 2 Million Products
author:
  login: maxime
  email: maxime@algolia.com
  display_name: maxime
  first_name: Maxime
  last_name: L
---

The following post is an interview of [Vincent Paulin][1], R&D Manager at [A Little
Market][2] (recently acquired by Etsy).

As a fast growing ecommerce site for handmade goods in France, A Little Market
has seen its marketplace grow from a few thousand to over 2 million products
in just 5 years. With 90,000 designers and artisans using A Little Market
marketplace to buy, sell and collaborate, search quickly became a major part
of their ecommerce strategy and user experience.

#### **![ALittleMarket][3]**

#### **What did you have in place as a search solution?**

"We implemented a Solr based search 5 years ago and had been trying to tweak
it to fit our growing needs.  We had selected this system for its flexibility,
however, over time, that flexibility translated into constant maintenance,
modifications and lower relevance in our search results.

Then we investigated Elasticsearch. It is complex, yet powerful. As I was
diving deeper into Elasticsearch I realized that I could quickly gain an "ok"
search experience; however, a powerful search experience would mean investing
more time than we had to configure it. Then I did a little math:  learning the
platform would take a few weeks, configuring servers - a few days, and
configuring and tuning semantic search perfectly - several months.

Then we found Algolia.  We only had 3 months and knew Algolia would be much
easier to implement, so we A/B tested everything to see how it would impact
the search experience.

#### **Can you tell us more about your integration process?**

The first thing we wanted to get done was to reference all the shops and our
best searches to make an autosuggest widget. Building this autosuggest with a
basic configuration took us 2 days.

Then we built an automatic task to aggregate shops and best searches every day
and configure Algolia indices. We also took on the task to create the front
javascript plugin. With the Algolia documentation and the examples on Github
it took us less than 1 hour.

The results of this first test were very encouraging.  With around 500k
requests per day, the response time was about 4 milliseconds on average and we
saw the conversion rate multiplied by 3 compared to the previous conversion
rate with a search bar with "no suggest". For A Little Mercerie, another
marketplace we manage, the improvement was about 4 times greater.

After this first test, we were ready to fully commit to Algolia for our whole
search experience. The first step was to create a script to index our entire
product database in Algolia. This was easy to do with batch insert in Algolia
indices. We selected some attributes of our products such as the title,
categories, materials and colors to be indexed. That was a first try. We
wanted it to be quick and simple.

With the help of the open source demo code we developed a full JS sandbox
which can display paginated results with faceting to show the progress to the
team.  In less than a week, we had a fully working sandbox and the results
were promising.  Our query time averaged less than 20 milliseconds on 2
millions records.  With confidence we started to upgrade the algorithm on
Algolia, test it, again and again, adding some attributes to index such as
specific events (christmas, valentine's day), custom tags, etc.

In addition, we implemented sorted results. They are really relevant with the
new numeric ranking option in settings. At that step we were able to sort
results by price, date, etc. You must create a specific index for each
specific ranking you need.  We also created a different index for each
language (French and Italian) and took this opportunity to do the same across
our  other websites, alittlemercerie.com and alittleepicerie.com.

To do this we created a custom API which abstracts the use of any kind of
search engine for all API clients. We end up losing the real-time search but
we need that for now in order to abstract everything and to collect data
before sending the results.

The next step was to erase the "no results" pages. For that, we were
progressively adding the last words of the query as optional words until we
had somes results.We never set as optional all the user queries.  We set at
least the first word or the first two words.

When search was ready, we still had plenty of time left to implement it on our
clients' applications. We took more time than was needed to implement Algolia.
The speed of iteration with the Algolia API enables us to test everything in a
much shorter timeframe.

#### **How has Algolia's API helped search on A Little Market?**

We are now able to answer more than 500/1000 requests per minute and we add
6000 new products every day to the search engine while over 3000 are removed,
in real time.

After this integration of the Algolia API, we saw an increase in our
conversion rate on search by 10%. This represents tens thousands of euros in
turnover per month for us. In a few weeks of work with one engineer, we had
replaced our main search engine for a better solution thanks to Algolia."


[1]: fr.linkedin.com/pub/vincent-paulin/71/1a3/86a
[2]: http://www.alittlemarket.com/
[3]: /algoliasearch-jekyll-hyde/assets/Capture-decran-2014-07-11-17.31.04-1024x486.png
