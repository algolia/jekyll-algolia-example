---
layout: post
title: Simplicity is the most Complex Feature!
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

I've been convinced for a long time that simplicity is the most important
property of a product. Long-gone are the 90s when a product was admired for
its complexity. But I am also convinced that this is the most complex property
to achieve and maintain as time passes by.

A good example of an over-complex product is Atlassian JIRA, a bug tracker
that also do scrum management and plenty of other things via dozens of
plugins. It's basically a toolbox to create the bug tracker adapted to your
company.

In my previous job, I faced an uncomfortable situation with JIRA because of
its complexity. We used it for bug tracking and scrum management and I tried
to upgrade our old version to the latest one. After some long hours to upgrade
our setup on a test server, I finally got the latest version working but most
of our installed plugins were not available anymore because the authors did
not port their plugins to the new plugin API. Of course each plugin was there
for a reason and I was in a tricky situation: keep the old version with
security issues or upgrade to a new version without our plugins.

But it was far more vicious: There were about 10 versions between our old
version and the latest one, and I didn't find any of these versions working
with our set of plugins! In the end, we were forced to keep our old version.

Atlassian forgot the most important lesson, even with a toolbox: simplicity!
This is probably more expensive for them to keep plugin backward
compatibility, but I would prefer for them to not have any plugin rather than
breaking compatibility at each release. The final system is too complex to be
maintainable and our final decision was to stop paying for JIRA support since
we were blocked with an old release. It is even bad for their business.

You should be focused on simplicity for your users even it this results in
more complexity for you (like maintaining backward compatibility)! As this
post is strongly related to backward compatibility of API, I encourage you to
reread this famous post of Jeol Spolsky: [How microsoft lost the API
war][1].


[1]: http://www.joelonsoftware.com/articles/APIWar.html
