---
layout: post
title: Improving Search for Twitter Handles
author:
  login: sylvain
  email: sylvain@algolia.com
  display_name: Sylvain Utard
  first_name: Sylvain
  last_name: Utard
---

Hello Twitter,

I have been using your service for awhile, and I love it!

At first, I was skeptical about what you could offer: Broadcasting to all my
friends that I was eating a pizza, or taking a walk, is not really my cup of
tea. But 3 years ago I figured out what Twitter was really meant for and how
it could help me in a totally different way from what I first thought:

  * sharing interesting articles,
  * checking if /replace by the service provider you want/ is down,
  * or catching up on HackerNews.

More recently, I discovered you had a feature that could help me even more: I
can now ask for support by tweeting. Tweeting is often faster and more
productive than sending an email. You taught me to include the recipient's
Handle in my tweets, and your current Handle auto-completion implementation
works pretty well: but what if you could provide a **better typo-tolerance and
ranking**? (I'm NOT speaking about your official OSX/iOS native clients and
its [totally unusable auto-completion feature][1]... btw, could you explain me why it
is different from the one on your website?).

I have been leading a search-engine development team over the last 5 years and
I'm now VP of engineering at Algolia. I am aware that considering my job, I
have kind of an "expert" point of view about search. But search has become so
essential that I am convinced it __must__ be irreproachable. Did you know
that 1.7M+ people are currently following

expecting great things from your search-engine, Twitter :) Here is how I would
improve search for Twitter handles:

For example, it would be nice if I could find President
[@barackobama][2] with his last name:

[![Search for Twitter handles including @obama yields less-than-stellar
results.][3]](https://blog.algolia.com/wp-
content/uploads/2013/12/Screen-Shot-2013-12-23-at-11.40.09.png)

Same for Justin:

[![Search for Twitter handles that could be Justin Bieber's yields less-than-
stellar results.][4]](https://blog.algolia.com/wp-
content/uploads/2013/12/Screen-Shot-2013-12-23-at-11.42.01.png)

Typo-tolerance is now a must-have, especially because we're all using
smartphones and tablets:

[![Search for Twitter handles should have typo tolerance.][5]](https://blog.algolia.com/wp-
content/uploads/2013/12/Screen-Shot-2013-12-23-at-11.38.19.png)

More and more handles are now prefixed/suffixed by "official", which makes
finding [@OfficialAdele][6] just impossible:

[![Search for Twitter handles that start with @official is broken.][7]](https://blog.algolia.com/wp-content/uploads/2013/12/Screen-Shot-2013-12-23-at-11.47.52.png)

**For sure we can improve it, let's code!**

First of all Twitter, I need your Handles database :)

  * I used your [Streaming API][8] to crawl about 20M+ accounts in ~2 weeks: it's not blazing fast but I must admit it does the job (and it's free). That's about 5 lines of Ruby with [TweetStream][9], good job guys!
  * and [Daemonize][10] to create a bin/crawler executable.
    
```ruby
#! /usr/bin/env ruby

require File.expand_path(File.join(File.dirname(__FILE__), '..', 'config', 'environment'))

daemon = TweetStream::Daemon.new('crawler', :log_output => true)
daemon.on_inited do
  ActiveRecord::Base.connection.reconnect!
  ActiveRecord::Base.logger = Logger.new(File.join(Rails.root, 'log/stream.log'), 'w+')
end
daemon.on_error do |message|
  puts "Error: #{message}"
end
daemon.sample do |status|
  Handle.create_from_status(status)
end
```

For each new tweet you send to me, I store the author (name + screen_name +
description + followers_count) and all his/her user mentions.

    
```ruby
class Handle < ActiveRecord::Base

  def self.create_from_user(user)
    h = Handle.find_or_initialize_by(screen_name: user.screen_name)
    puts h.screen_name if h.new_record?
    h.name = user.name
    h.description = (user.description || "")[0..255]
    h.followers_count = user.followers_count
    h.updated_at ||= DateTime.now
    h.save
    h
  end

  def self.create_from_status(status)
    Handle.create_from_user(status.user)
    status.user_mentions.each do |mention|
      m = Handle.find_or_initialize_by(screen_name: mention.screen_name)
      m.updated_at ||= DateTime.now
      m.name = mention.name
      m.mentions_count ||= 0
      m.mentions_count += 1
      m.save
    end
  end

end
```

And every minute, I re-index the last-updated accounts with a batch request
using [algoliasearch-rails][11],

    
```ruby
every 1.minute, roles: [:cron] do
  runner "Handle.where('updated_at >= ?', 1.minute.ago).reindex!"
end
```

The result order is based on several criteria:

  * the number of typos,
  * the matching attributes: the name/handle is more important than the description,
  * the proximity between matched words,
  * and the followers count (I also use the "mentions count" if my crawler didn't get the followers count yet).

I could have improved the results by using the user's list of
followers/following but I was limited by your [Rate
Limits][12]. **Instead, I chose to
emphasize your top-users **(accounts having 10M+ followers).

Here is the configuration I used

    
```ruby
class Handle < ActiveRecord::Base

  include AlgoliaSearch
  algoliasearch per_environment: true, auto_index: false, auto_remove: false do
    # add an extra score attribute
    add_attribute :score

    # add an extra full_name attribute: screen_name + name
    add_attribute :full_name

    # do not take `full_name`'s words order into account, `full_name` is more important than `description`
    attributesToIndex ['unordered(full_name)', :description]

    # list of attributes to highlight
    attributesToHighlight [:screen_name, :name, :description]

    # use followers_count OR mentions_count to sort results (last sort criteria)
    customRanking ['desc(score)']

    # @I_love_you
    separatorsToIndex '_'

    # tag top-users
    tags do
      followers_count > 10000000 ? ['top'] : []
    end
  end

  def full_name
    # consider screen_name and name equal
    # the name should not match exact so we concatenate it with the screen_name
    [screen_name, "#{screen_name} #{name}"]
  end

  # the custom score
  def score
    return followers_count if followers_count > 0
    if mentions_count < 10
      mentions_count
    elsif mentions_count < 100
      mentions_count * 10
    elsif mentions_count < 1000
      mentions_count * 100
    else
      mentions_count * 1000
    end
  end

end
```

The user query is composed by 2 backend queries:

  * the first one retrieves all matching top-users (could be replaced by a query targeting your followers/following only)
  * the second one the others.

[**Try it for yourself**][13], and enjoy
relevant and highlighted results after the first keystroke: [Twitter Handles
Search][14].


[1]: http://blog.algolia.com/why-autocomplete-in-twitter-on-mobile-sucks/
[2]: https://twitter.com/barackobama
[3]: /algoliasearch-jekyll-hyde/assets/Screen-Shot-2013-12-23-at-11.40.09-263x300.png
[4]: /algoliasearch-jekyll-hyde/assets/Screen-Shot-2013-12-23-at-11.42.01-262x300.png
[5]: /algoliasearch-jekyll-hyde/assets/Screen-Shot-2013-12-23-at-11.38.19-263x300.png
[6]: https://twitter.com/officialadele
[7]: /algoliasearch-jekyll-hyde/assets/Screen-Shot-2013-12-23-at-11.47.52.png
[8]: https://dev.twitter.com/docs/streaming-apis
[9]: https://github.com/tweetstream/tweetstream
[10]: https://github.com/bmc/daemonize
[11]: https://github.com/algolia/algoliasearch-rails
[12]: https://dev.twitter.com/docs/rate-limiting/1.1
[13]: http://twittersearch.algolia.io/
[14]: http://twittersearch.algolia.io/
