---
layout: post
title: 'Realtime Search: Security and our Javascript Client'
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

_**Edit: As suggested on [Hacker
News][1], SHA256 is not secure, as
it allows a length extension attack. We have replaced it with HMAC-SHA256.**_

Instant is in our DNA, so our first priority was to build a search backend
that would be able to return relevant realtime search results in a few
milliseconds. However, the backend is just one variable in our realtime
equation. The response time perceived by the end user is the total lapse of
time between their first keystroke and the final display of their results.
Thus, with an extremely fast backend, solving this equation comes down to
optimising network latency. This is an issue we solve in two steps:

  * First, we have [datacenters in three different locations][2], allowing us to answer queries in North America, Europe and Asia in less than 100ms (including search computation).
  * Second, to keep reducing this perceived latency, queries must be sent directly from the end users' browsers or mobile phones to our servers. To avoid intermediaries like your own servers, we offer a JavaScript client for websites and ObjC/Android/C# clients for mobile apps.

## The security challenge of JavaScript

Using this client means that you need to include an API key in your JavaScript
(or mobile app) code. The first security issue with this approach is that this
key can be easily retrieved by anyone who simply looks at the code of the
page. This gives that person the potential to modify the content behind the
website/mobile application! To fix this problem, we provide search-only API
keys which protect your indexes from unauthorized modifications.

This was a first step and we've quickly had to solve two other security
issues:

  * **Limiting the ability to crawl your data: **you may not want people to get all your data by continuous querying. The simple solution was to limit the number of API calls a user could perform in a given period of time. We implemented this by setting a rate limit per IP address. However, this approach is not acceptable if a lot of users are behind a global firewall, thus sharing one IP address. This is very likely for our corporate users.
  * **Securing access control**:  you may need to restrict the queries of a user to specific content. For example, you may have power users who should get access to more content than "regular" users. The easy way to do it is by using filters. The problem here with simple filters in your JavaScript code is that people can figure out how to modify these filters and get access to content they are not be supposed to see.

## How we solve it altogether

Today, most websites and applications require people to create an account and
log in to access a personalized experience (think of CRM applications,
Facebook or even Netflix). We decided to use these user IDs to solve these two
issues by creating signed API keys. Let's say you have an API key with search
only permission and want to apply a filter on two groups of content (public OR
power_users_only) for a specific user (id=42):

    
    api_key=20ffce3fdbf036db955d67645bb2c993
    query_filters=(public,power_users_only)
    user_token=42

You can generate a secured API key in your backend that is defined by a hash
(HMAC SHA 256) of three elements:

    
    secured_api_key=HMAC_SHA_256(api_key, query_filters + user_token)
    secured_api_key=HMAC_SHA_256("20ffce3fdbf036db955d67645bb2c993", "(public,power_users_only)" + "42")
    secured_api_key="3abb95c273455ce9b57c61ee5258ba44093f17022dd4bfb39a37e56bee7d24a5"

For example, if you are using rails, the code in your backend would be:

    
    secured_key = Algolia.generate_secured_api_key('20ffce3fdbf036db955d67645bb2c993', '(public,power_users_only)', '42')

You can then initialize your JavaScript code with the secured API key and
associated information:

The user identifier (defined by SetUserToken) is used instead of the IP
address for the rate limit and the security filters (defined by
SetSecurityTags) are automatically applied to the query.

In practice, if a user wants to overstep her rights, she will need to modify
her security tags and figure out the new hash. Our backend checks if a query
is legit by computing all the possible hashes using all your available API
keys for the queried index, as well as the security tags defined in the query
and the user identifier (if set).  If there is no match between the hash of
the query and the ones we computed, we will return a permission denied (403).
Don't worry, reverse-engineering the original API key using brute-force would
require years and [thousands of
core][3].

You may want to apply security filters without limiting the rate of queries,
so if you don't need both of these features, you can use only one.

We launched this new feature a few weeks ago and we have received very good
feedback so far. Our customers don't need to choose anymore between security
and realtime search. If you see any way to improve this approach, we would
love to hear your feedback!


[1]: https://news.ycombinator.com/item?id=7419205
[2]: http://blog.algolia.com/added-asian-datacenter-offer/
[3]: http://en.wikipedia.org/wiki/SHA-2#Comparison_of_SHA_functions
