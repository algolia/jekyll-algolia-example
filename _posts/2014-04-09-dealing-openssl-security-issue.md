---
layout: post
title: Dealing with OpenSSL Heartbleed Vulnerability
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

Yesterday, the OpenSSL project released an update to fix a serious security
issue. This vulnerability was disclosed in [CVE-2014-0160](https://web.nvd.nis
t.gov/view/vuln/detail?vulnId=CVE-2014-0160) and is more widely known as the
[Heartbleed vulnerability](http://heartbleed.com/). It allows an attacker to
grab the content in memory on a server. Given the widespread use of OpenSSL
and the versions affected, this vulnerability affects a large percentage of
services on the internet.

Once the exploit was revealed, we responded immediately: All Algolia services
were secured the same day, by 3pm PDT on Monday, April 7th. The fix was
applied on all our API servers and our website. We then generated new SSL
certificates with a new private key.

Our website is also dependent on Amazon Elastic Load Balance, which was
affected by this issue and [updated](http://aws.amazon.com/security/security-
bulletins/aws-services-updated-to-address-openssl-vulnerability/) later on
Tuesday, April 8th. We then changed the website certificate.

**All Algolia servers are no longer exposed to this vulnerability.**

## Your credentials

We took the time to analyze the past activity on our servers and did not find
any suspicious activity. We are confident that no credentials were leaked.
However, given that this exploit existed in the wild for such a long time, it
is possible that an attacker could have stolen API keys or passwords without
our knowledge. As a result, we recommend that all Algolia users change the
passwords on their accounts. We also recommend that you reset your Algolia
administration API key, which you can do at the bottom of the "Credential"
section in your dashboard. Be careful to update it everywhere you use it in
your code (once you have patched your SSL library if you too are vulnerable).

## Security at Algolia

The safety and security of our customer data are our highest priorities. We
are continuing to monitor the situation and will respond rapidly to any other
potential threats that may be discovered.

If you have any questions or concerns, please email us directly at
[security@algolia.com](mailto:security@algolia.com)

