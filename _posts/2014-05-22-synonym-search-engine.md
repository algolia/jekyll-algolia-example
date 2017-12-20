---
layout: post
title: A New Way to Handle Synonyms in a Search Engine
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

We recently added the support for Synonyms in Algolia! It has been the most
requested feature in Algolia since our launch in September. While it may seem
simple, it actually took us some time to implement because we wanted to do it
in a different way than classic search engines.

## What's wrong with synonyms

There are two main problems with how existing search engines handle synonyms.
These issues disturb the user experience and could make them think _"this
search engine is buggy"_.

### Typeahead

In most search engines, synonyms are not compatible with typeahead search. For
example, if you want tablet  to equal  ipad in a query, the prefix search for
t , ta , tab , tabl  & table  will not trigger the expansion on iPad ; Only
the tablet query will. Thus, a single new letter in the search bar could
totally change the result set, catching users off-guard.

### Highlighting

Highlighting matched text is a key element of the user experience, especially
when the search engine tolerates typos. This is the difference between making
users think _"I don't understand this result"_ and _"This engine was able to
understand my errors"_. Synonym expansions are rarely highlighted, which
breaks the trust of the users in the search results and can feel like a bug.

## Our implementation

We have identified two different use cases for synonyms: equalities and
placeholders. The first and most common use case is when you tell the search
engine that several words must be considered equal, for example st and street
in an address. The second use case, which we call a _placeholder_, is when you
indicate that a specific token can be replaced by a set of possible words and
that the token itself is not searchable. For example, the content <number>
street could be matched by the queries 1st street or 2nd street but not the
query number street.

For the first use case, we have added a support of synonyms that is compatible
with prefix search and have implemented two different ways to do highlighting
(controlled by thereplaceSynonymsInHighlight  query parameter):

  1. A mode where the original word that matched via a synonym is highlighted. For example if you have a record that contains black ipad 64GB  and a synonym black equals dark, then the following queries will fully highlight the black word : ipad d , ipad da , ipad dar & ipad dark. The typeahead search is working and the synonym expansion is fully highlighted: `**black** **ipad** 64GB` .
  2. A mode where the original word is replaced by the synonym, and the matched prefix is highlighted. For example ipad d  query will replace black by dark and will highlight the first letter of dark: `**d**ark **ipad** 64GB`. This method allows to fully explain the results when the original word can be safely replaced by the matched synonym.

For the second use case, we have added support for placeholders. You can add a
specific token in your records that will be safely replaced by a set of words
defined in your configuration. The highlighting mode that replaces the
original word by the expansion totally makes sense here. For example if you
have <streetnumber> mission street  record with a placeholder <streetnumber> =
[ "1st", "2nd", ....] , then the query 1st missionstreet will replace <number>
by 1st  and will highlight all words: `**1st mission street**`.

We believe this is a better way to handle synonyms and we hope you will like
it :) We would love to get your feedback and ideas for improvement on this
feature! Feel free to contact us at **hey(at)algolia.com**.

