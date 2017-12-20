---
layout: post
title: 'Hacker News search: 6.5 million articles and comments at your fingertips'
author:
  login: sylvain
  email: sylvain@algolia.com
  display_name: Sylvain Utard
  first_name: Sylvain
  last_name: Utard
---

We are [Hacker News][1] readers and probably just
like you, there is not a day that goes by we don't use it. It is a little like
checking the weather app of the tech world. Long story short, Hacker News is
awesome, and we wanted to add our two cents to make it even greater to use.

Indeed, here is our problem: how do we instantly access the old posts we wish
we had saved?

## Powering a new Hacker News search engine

Up until now we've been using [hnsearch.com][2],
maintained for years by the great folks at [Octopart][3]. I
hope we speak on behalf of the HN community here, we are all grateful for the
work they put in hnsearch.com and they inspired us to pursue their effort.

Back in September 2013, we created a "[homemade Hacker News
crawler][4]" and built a search
engine with the data we could get. It was not perfect but somehow, it did the
job fine.

[Now part of the Ycombinator W14 batch](http://blog.ycombinator.com/algolia-
yc-w14-launches-a-search-api-that-lets-you-provide-apple-spotlight-like-
realtime-search-for-your-app-or-service), we have a direct access to the data
and it has allowed us to provide instant search for the entire content of
Hacker News, 1.2 million articles, 5.2 million comments as of today. See for
yourself right here: [hn.algolia.com][5]

## Here is how we did it

  * ### Hacker News API access

    * YC provides us a private API access to fetch batches of 1000 items (an item being a comment or a post). Every two minutes, we update our database with the latest 1000 items. Last 48,000 items are refreshed every hour to keep the number of votes and comments up to date.
    
    ```
    # Yep, that's a Lisp API :)
    EXPORT_REGEXP = %r{^((d+) (story|comment|poll|pollopt) "(.+)" (d+) (?:nil|"(.*)") (?:nil|"(.+)") (?:nil|"(.*)") (?:nil|-?(d+)) (?:nil|(([d ]+))) (?:nil|(d+)))$}
    ```

  * ### Thumbnails generation

    * We use [wkhtmltoimage][6] to render the URLs and generate the associated thumbnails. Playing with connection timeouts and JavaScript infinite loops was a pleasure:
    
    ```
    (timeout 60 xvfb-run --auto-servernum --server-args="-screen 0, 1024x768x24" 
    wkhtmltoimage-amd64 --height 768 --use-xserver--javascript-delay 30000 "$URL" "$FILE" || 
    timeout 60 xvfb-run --auto-servernum --server-args="-screen 0, 1024x768x24" 
    wkhtmltoimage-amd64 --height 768 --use-xserver --disable-javascript "$URL" "$FILE") && 
    convert "$FILE" -resize '100!x100' "$FILE"
    ```

  * ### Thumbnails storage

    * Thumbnails are resized and stored on a S3 bucket.
    
    ```
    AWS::S3::S3Object.store("#{id}.png", open(temp_file), 'hnsearch', access: :public_read)
    ```

  * ### Thumbnails distribution

    * We configured a CloudFront instance targeting the S3 bucket to distribute thumbnails with low latency and high data transfer speed. We followed Amazon's associated [developer guide][7].
  * ### Indexing

    * We used the "[algoliasearch-rails][8]" gem and a standard (Ruby on Rails) MySQL-backed ActiveRecord setup. Indexing is performed automatically as soon as new items are added to the database, providing a near-realtime experience.
  * ### Configuration
    
    ```ruby
    class Item < ActiveRecord::Base
      include AlgoliaSearch
    
      algoliasearch per_environment: true do
        # the list of attributes sent to Algolia's API
        attribute :created_at, :title, :url, :author, :points, :story_text, :comment_text, :author, :num_comments, :story_id, :story_title, :story_url
        attribute :created_at_i do
          created_at.to_i
        end
    
        # The order of the attributes sets their respective importance.
        # `title` is more important than `{story,comment}_text`, `{story,comment}_text` more than `url`, `url` more than `author`
        # btw, do not take into account position to avoid first word match boost
        attributesToIndex ['unordered(title)', 'unordered(story_text)', 'unordered(comment_text)', 'unordered(url)', 'author', 'created_at_i']
    
        # add tags used for filtering
        tags do
          [item_type, "author_#{author}", "story_#{story_id}"]
        end
    
        # Custom ranking allows to automatically sort the results by a custom criteria
        # in this case, a decreasing sort of the number of HN points and comments.
        customRanking ['desc(points)', 'desc(num_comments)']
    
        # controls the way results are sorted sorting on the following 4 criteria (one after another)
        # I removed the 'exact' match critera (improve 1-words query relevance, doesn't fit HNSearch needs)
        ranking ['typo', 'proximity', 'attribute', 'custom']
    
        # google+, $1.5M raises, C#: we love you
        separatorsToIndex '+#$'
      end
    
      def story_text
        item_type_cd != Item.comment ? text : nil
      end
    
      def story_title
        comment? && story ? story.title : nil
      end
    
      def story_url
        comment? && story ? story.url : nil
      end
    
      def comment_text
        comment? ? text : nil
      end
    
      def comment?
        item_type_cd == Item.comment
      end
    
      def num_comments
        item_type_cd == Item.story ? story_comments.count : nil
      end
    end
    ```

  * ### Search

    * Queries are sent directly to our API via the[ javascript client][9], the javascript code uses a public API-Key that can only perform queries.

## Seeking feedback from the community

There is still room for improvement and we would love to know how you are
searching for news on HN. What is important for you? Are you searching by
date, by upvote, by comment or by user? All together maybe?

We would love to have your feedback! Don't hesitate to checkout the code: [We
open-sourced it][10].

Special thanks to the [Octopart][11] and
[YC][12] teams for making this experience possible!

Give it a try now: [hn.algolia.com][13]


[1]: https://news.ycombinator.com
[2]: http://www.hnsearch.com
[3]: http://octopart.com/
[4]: https://news.ycombinator.com/item?id=6476003
[5]: http://hn.algolia.com/
[6]: https://code.google.com/p/wkhtmltopdf/
[7]: http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/MigrateS3ToCloudFront.html
[8]: https://github.com/algolia/algoliasearch-rails
[9]: https://github.com/algolia/algoliasearch-client-js
[10]: https://github.com/algolia/hn-search
[11]: http://octopart.com/
[12]: http://ycombinator.com/
[13]: http://hn.algolia.com
