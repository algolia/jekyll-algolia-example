---
layout: post
title: Why JSONP is still Mandatory
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

At Algolia, we are convinced that search queries need to be sent directly from
the browser (or mobile app) to the search-engine in order to have a realtime
search experience. This is why we have developed a search backend that replies
within a few milliseconds through an API that [handles
security](http://blog.algolia.com/handle-security-realtime-search/) when
called from the browser.

## Cross domain requests

For security reasons, the default behavior of a web browser is to block all
queries that are going to a domain that is different from the website they are
sent from. So when using an external HTTP-based search API, all your queries
should be blocked because they are sent to an external domain. There are two
methods to call an external API from the browser:

###  JSONP

The [JSONP](http://en.wikipedia.org/wiki/JSONP) approach is a workaround that
consists of calling an external API  with a DOM `<script>`  tag. The `<script>`
tag is allowed to load content from any domains without security restrictions.
The targeted API needs to expose a HTTP GET endpoint and return Javascript
code instead of the regular JSON data. You can use this jQuery code to
dynamically call a JSONP URL:

    
```javascript
$.getJSON( "http://api.algolia.io/1/indexes/users?query=test", function( data ) { .... }
```

In order to retrieve the API answer from the newly included JavaScript code,
jQuery automatically appends a callback argument to your URL (for example
&callback=method12 ) which must be called by the JavaScript code that your API
generates.

This is what a regular JSON reply would look like:

    
```json
{
  "results": [ ...]
}
```

Instead, the JSONP-compliant API generates:

    
```javascript
method12({"results": [ ...]});
```

### Cross Origin Resource Sharing

[CORS](http://en.wikipedia.org/wiki/Cross-origin_resource_sharing) (Cross
Origin Resource Sharing) is the proper approach to perform a call to an
external domain. If the remote API is CORS-compliant, you can use a regular
XMLHttpRequest  JavaScript object to perform the API call. In practice the
browser will first perform an HTTP OPTIONS request to the remote API to check
which caller domains are allowed and if it is authorized to execute the
requested URL.

For example here is a CORS request issued by a browser. The most important
lines are the last two headers that specify which permissions are checked. In
this case, the method is POST and the three specific HTTP headers that are
requested.

    
    OPTIONS http://latency.algolia.io/1/indexes/*/queries
    > Host: latency.algolia.io
    > Origin: http://demos.algolia.com
    > Accept-Encoding: gzip,deflate,sdch
    > Accept-Language: en-US,en;q=0.8,fr;q=0.6
    > User-Agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_9_2)
    > Accept: */*
    > Referer: http://demos.algolia.com/eventbrite/
    > Connection: keep-alive
    > Access-Control-Request-Headers: x-algolia-api-key, x-algolia-application-id, content-type
    > Access-Control-Request-Method: POST

The server reply will be similar to this one:

    
    < HTTP/1.1 200 OK
    < Server: nginx/1.6.0
    < Date: Tue, 13 May 2014 08:33:55 GMT
    < Content-Type: text/plain
    < Content-Length: 0
    < Connection: keep-alive
    < Access-Control-Allow-Origin: *
    < Access-Control-Allow-Methods: GET, PUT, DELETE, POST, OPTIONS
    < Access-Control-Allow-Headers: x-algolia-api-key, x-algolia-application-id, content-type
    < Access-Control-Allow-Credentials: false < Expires: Wed, 14 May 2014 08:33:55 GMT
    < Cache-Control: max-age=86400
    < Access-Control-Max-Age: 86400

This answer indicates that this POST method can be called from any domain
(Access-Control-Allow-Origin: \* ) and with the requested headers.

CORS has many advantages. First, it allows access to a real REST API with all
HTTP verbs (mainly GET, POST, PUT, DELETE) and it also allows to better handle
errors in an API (bad requests, object not found, ...). The major drawback is
that it is only supported by modern browsers (Internet Explorer ≥ 10, Firefox
≥ 3.5, Chrome ≥ 3, Safari ≥ 4 & Opera ≥ 12; Internet Explorer 8 & 9 provides
partial support via theXDomainRequest  object).

## Our initial conclusion

Because of the advantages of CORS in terms of error handling, we started with
a CORS implementation of our API. We also added a specific support for
Internet Explorer 8 & 9 using the  XDomainRequest  JavaScript object (they do
not support XMLHttpRequest). The main difference is that XDomainRequest  does
not support HTTP headers so we added another way to specify user credentials
in the body of the POST request (it was initially only supported via HTTP
headers).

We were confident that we were supporting almost all browsers with this
implementation, as only very old browsers could cause problems. But we were
wrong!

## CORS problems

The reality is that CORS still causes problems, even with modern browsers. The
biggest problem we have found was with some firewalls/proxies that refuse HTTP
OPTIONS queries. We even found software on some computers that were blocking
CORS requests, as the [Cisco AnyConnect VPN
client](http://www.bennadel.com/blog/2559-cisco-anyconnect-vpn-client-may-
block-cors-ajax-options-requests.htm), which is widely used in the enterprise
world. We have found this issue when a TechCrunch employee was not able to
operate search on [crunchbase.com](http://www.crunchbase.com) because the
AnyConnect VPN client was installed on his laptop.

Even in 2014 with a large majority of browsers supporting CORS, it is not
possible to have perfect service quality with a CORS-enabled REST API!

## The solution

Using JSONP is the only solution to ensure great compatibility with old
browsers and handle problems with a misconfigured firewall/proxy. However,
CORS offers the advantage of proper error-handling, so we do not want to limit
ourselves to JSONP.

In the latest version of our JavaScript client, we decided to use CORS with a
fallback on JSONP. At client initialization time, we check if the browser
supports CORS and then perform an OPTIONS query to check that there is no
firewall/proxy that blocks CORS requests. If there is any error we fallback on
JSONP. All this logic is available in our JavaScript client without any
API/code change for our customers.

Having CORS support with automatic fallback on JSONP is the best way we have
found to ensure great service quality and to support all corner case
scenarios. If you see any other way to do it, your feedback is very welcome.

