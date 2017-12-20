---
layout: post
title: Github Awesome Autocomplete browser extension for Chrome and Firefox
author:
  login: sylvain
  email: sylvain@algolia.com
  display_name: Sylvain Utard
  first_name: Sylvain
  last_name: Utard
---

By working every day on building the best search engine, we've become obsessed
with our own search experience on the websites and mobile applications we use.

We're git addicts and love using GitHub to store every single idea or project
we work on. We use it both for our private and public repositories ([12 API
clients][1], [HN
Search][2] or various
[d][3]
[e][4] [m][5] [o][6]
[s][7]. We use every day its search
function and we decided to re-build it the way we thought it should be.  We're
proud to share it with the community via this [Chrome
extension][8]. Our Github Awesome Autocomplete
enables a seamless and fast access to GitHub resources via an as-you-type
search functionality.

## Install your Christmas Gift now!

[![Github Awesome Autocomplete Algolia Search][9]](https://chrome.google.com/webstore/detail/github-awesome-
autocomple/djkfdjpoelphhdclfjhnffmnlnoknfnd)

## Features

The Chrome extension replaces GitHub's search bar and add autocomplete
capabilities on:

  * top public repositories

  * last active users

  * your own private repositories (this one is done locally in JavaScript without Algolia: the list of private repositories remains locally in your browser)

![][10]

## How does it work?

We continuously retrieve the most watched repositories and the last active
users using [GitHub Archive][11] dataset. Users and
repositories are stored in 2 Algolia indices: users and repositories. The
queries are performed using [our JavaScript API
client][12] and the
autocomplete menu is based on Twitter's
[typeahead.js][13] library.

The underlying Algolia account is replicated in 6 regions using our
[DSN][14] feature, answering every query in 50-100ms
wherever you are (network latency included!). Regions include US West, US
East, Europe, Singapore, Australia & India.

## Exporting the records from GitHub Archive

We used GitHub's Archive dataset to export top repositories and last active
users using Google's BigQuery:

    
    ;; export repositories
    SELECT
      a.repository_name as name,
      a.repository_owner as owner,
      a.repository_description as description,
      a.repository_organization as organization,
      a.repository_watchers AS watchers,
      a.repository_forks AS forks,
      a.repository_language as language
    FROM [githubarchive:github.timeline] a
    JOIN EACH
      (
         SELECT MAX(created_at) as max_created, repository_url
         FROM [githubarchive:github.timeline]
         GROUP EACH BY repository_url
      ) b
      ON 
      b.max_created = a.created_at and
      b.repository_url = a.repository_url
    
    
    ;; export users
    SELECT
      a.actor_attributes_login as login,
      a.actor_attributes_name as name,
      a.actor_attributes_company as company,
      a.actor_attributes_location as location,
      a.actor_attributes_blog AS blog,
      a.actor_attributes_email AS email
    FROM [githubarchive:github.timeline] a
    JOIN EACH
      (
         SELECT MAX(created_at) as max_created, actor_attributes_login
         FROM [githubarchive:github.timeline]
         GROUP EACH BY actor_attributes_login
      ) b
      ON 
      b.max_created = a.created_at and
      b.actor_attributes_login = a.actor_attributes_login
    

## Configuring Algolia indices

Here are the 2 index configurations we used to build the search:

### Repositories

![][15]

### Users

![][16]

###

## Want to contribute?

It's open-source and we'll be happy to get your feedback! Just use [GitHub's
issues][17] to
report any idea you have in mind. We also love pull-requests :)

Source code: [https://github.com/algolia/github-awesome-
autocomplete](https://github.com/algolia/github-awesome-autocomplete)

Install it now: [Github Awesome Autocomplete on Google Chrome Store
[FREE][18]]

## Or just want to add an instant search in your website / application?

Feel free to create a 14-days FREE trial at
[http://www.algolia.com](http://www.algolia.com/) and follow one of our step
by step tutorials at
[https://www.algolia.com/doc/tutorials](https://www.algolia.com/doc/tutorials)


[1]: https://www.algolia.com/doc/apiclients
[2]: https://github.com/algolia/hn-search
[3]: https://github.com/algolia/instant-search-demo
[4]: https://github.com/algolia/facebook-search
[5]: https://github.com/algolia/linkedin-search
[6]: https://github.com/algolia/meetup-search
[7]: https://github.com/algolia/twitter-search)
[8]: https://chrome.google.com/webstore/detail/github-awesome-autocomple/djkfdjpoelphhdclfjhnffmnlnoknfnd
[9]: /algoliasearch-jekyll-hyde/assets/ChromeWebStore_BadgeWBorder_v2_206x58.png
[10]: /algoliasearch-jekyll-hyde/assets/hackpad.com_Y3thzadEtdY_p.233467_1419338581444_capture.gif
[11]: http://www.githubarchive.org/
[12]: https://github.com/algolia/algoliasearch-client-js
[13]: http://twitter.github.io/typeahead.js/
[14]: https://www.algolia.com/dsn
[15]: /algoliasearch-jekyll-hyde/assets/hackpad.com_Y3thzadEtdY_p.233467_1419338775543_Screen%20Shot%202014-12-23%20at%2013.46.04.png
[16]: /algoliasearch-jekyll-hyde/assets/hackpad.com_Y3thzadEtdY_p.233467_1419338826276_Screen%20Shot%202014-12-23%20at%2013.45.54.png
[17]: https://github.com/algolia/github-awesome-autocomplete/issues
[18]: https://chrome.google.com/webstore/detail/github-awesome-autocomple/djkfdjpoelphhdclfjhnffmnlnoknfnd
