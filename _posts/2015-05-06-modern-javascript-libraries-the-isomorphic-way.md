---
layout: post
title: 'Modern JavaScript libraries: the isomorphic way'
author:
  login: vvo
  email: vincent@algolia.com
  display_name: Vincent
  first_name: Vincent
  last_name: Voyer
---

Algolia's DNA is really about performance. We want our search engine to answer
relevant results as fast as possible.

To achieve the best end-to-end performance we've decided to go with JavaScript
since the total beginning of Algolia. Our end-users search using our REST API
directly from their browser - with JavaScript - without going through the
websites' backends.

Our JavaScript & Node.js API clients were implemented 2 years ago and were now
lacking of all modern best practices:

  * not following the [error-first](http://fredkschott.com/post/2014/03/understanding-error-first-callbacks-in-node-js/) or [callback-last](http://blog.izs.me/post/59142742143/designing-apis-for-asynchrony) conventions;
  * inconsistent API between the Node.js and the browser implementations;
  * no [Promise](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise) support;
  * Node.js module named [algolia-search](https://www.npmjs.com/package/algolia-search), browser module named [algoliasearch](https://www.npmjs.com/package/algoliasearch);
  * cannot use the same module in Node.js or the browser (obviously);
  * browser module could not be used with [browserify](http://browserify.org/) or [webpack](http://webpack.github.io/). It was exporting multiple properties directly in the window object.

This blog post is a summary of the three main challenges we faced while
modernizing our JavaScript client.

## tl;dr;

Now the good news: we have a new isomorphic [JavaScript API
client](http://github.com/algolia/algoliasearch-client-js).

> Isomorphic JavaScript apps are JavaScript applications that can run both
client-side and server-side.

The backend and frontend share the same code.

>

> [isomorphic.net](http://isomorphic.net/)

Here are the main features of this new API client:

  * works in [Node.js](https://nodejs.org/) 0.10, 0.12, [iojs](https://iojs.org/en/index.html) and from Internet Explorer 8 up to modern browsers;
  * has a [Promise + callback](https://github.com/algolia/algoliasearch-client-js#callback-convention) API;
  * is available at [npmjs.com/algoliasearch](https://www.npmjs.com/package/algoliasearch) and on [cdn.jsdelivr.net](http://cdn.jsdelivr.net/algoliasearch/3/algoliasearch.min.js);
  * has [builds](https://github.com/algolia/algoliasearch-client-js/tree/master/dist) for jQuery, AngularJS and Parse.com;
  * is compatible with all module loaders like [browserify](http://browserify.org/) or [webpack](http://webpack.github.io/);
  * is fully [tested](https://github.com/algolia/algoliasearch-client-js/tree/master/test) in all supported environments.

If you were using our previous libraries, we have migration guides for both
[Node.js](https://github.com/algolia/algoliasearch-client-
js/wiki/Node.js-v1.x.x-migration-guide) and [the
browser](https://github.com/algolia/algoliasearch-client-js/wiki/Migration-
guide-from-2.x.x-to-3.x.x).

#

# Challenge #1: testing

Before being able to merge the Node.js and browser module, we had to remember
how the current code is working. An easy way to understand what a code is
doing is to read the tests. Unfortunately, in the previous version of the
library, we had [only one test](https://github.com/algolia/algoliasearch-
client-js/tree/ba71a68005945c73f7157a8222a9c758e332eceb/test). One test was
not enough to rewrite our library. Let's go testing!

## Unit? Integration?

When no tests are written on a library of [~1500+
LOC](https://github.com/algolia/algoliasearch-client-
js/blob/ba71a68005945c73f7157a8222a9c758e332eceb/src/algoliasearch.js#L1484),
what are the tests you should write first?

Unit testing would be too close to the implementation. As we are going to
rewrite a lot of code later on, we better not go too far on this road right
now.

Here's the flow of our JavaScript library when doing a search:

  * initialize the library with `algoliasearch()`
  * call `index.search('something', callback)`
  * browser issue an HTTP request
  * `callback(err, content)`

From a testing point of view, this can be summarized as:

  * input: method call
  * output: HTTP request

Integration testing for a JavaScript library doing HTTP calls is interesting
but does not scale well.

Indeed, having to reach Algolia servers in each test would introduce a shared
testing state amongst developers and continuous integration. It would also
have a slow TDD feedback because of heavy network usage.

Our strategy for testing our JavaScript API client was to mock (do not run
away right now) the [XMLHttpRequest](https://xhr.spec.whatwg.org/) object.
This allowed us to test our module as a black box, providing a good base for a
complete rewrite later on.

This is not unit testing nor integration testing, but in between. We also
planned in the coming weeks on doing a separate full integration testing suite
that will go from the browser to our servers.

## faux-jax to the rescue

Two serious candidates showed up to help in testing HTTP request based
libraries

  * [pgte/nock ](https://github.com/pgte/nock)
  * and [Sinon.js fake XMLHttpRequest](http://sinonjs.org/docs/#server).

Unfortunately, none of them met all our requirements. Not to mention, the
AlgoliaSearch JavaScript client had a really smart failover request strategy:

  * use [XMLHttpRequest](https://xhr.spec.whatwg.org/) for browsers supporting CORS,
  * or use [XDomainRequest](https://msdn.microsoft.com/en-us/library/ie/cc288060%28v=vs.85%29.aspx) for [IE < 10](http://caniuse.com/#feat=cors),
  * or use [JSONP](https://blog.algolia.com/jsonp-still-mandatory/) in situations where none of the preceding is available.

This seems complex but we really want to be available and compatible with
every browser environment.

  * **Nock** works by mocking calls to the Node.js [http module](https://nodejs.org/api/http.html), but we directly use the XMLHttpRequest object.
  * **Sinon.js** was doing a good job but [was lacking](https://github.com/cjohansen/Sinon.JS/pull/600#issuecomment-74000915) some [XDomainRequest](https://msdn.microsoft.com/en-us/library/ie/cc288060%28v=vs.85%29.aspx) feature detections. Also it was really tied to Sinon.js.

As a result, we created [algolia/faux-jax](https://github.com/algolia/faux-
jax). It is now pretty stable and can mock XMLHttpRequest, XDomainRequest and
even http module from Node.js. It means **faux-jax** is an isomorphic HTTP
mock testing tool. It was not designed to be isomorphic. It was easy to add
the Node.js support thanks to [moll/node-mitm](https://github.com/moll/node-
mitm).

## Testing stack

The testing stack is composed of:

  * [substack/tape](https://github.com/substack/tape/), isomorphic testing and assertion framework
  * [defunctzombie/zuul](https://github.com/defunctzombie/zuul/), local and continuous integration test runner
  * [algolia/faux-jax](https://github.com/algolia/faux-jax), isomorphic HTTP mocking library

The fun part is done, now onto the tedious one: **writing tests**.

## Spliting tests cases

We divided our tests in two categories:

  * **simple test cases**: check that an API command will generate the corresponding HTTP call
  * **advanced tests**: timeouts, keep-alive, JSONP, request strategy, DNS fallback, ..

### Simple test cases

Simple test cases were written as [table driven
tests](https://github.com/golang/go/wiki/TableDrivenTests):

[![It's a simple
JavaScript file, exporting test cases as an
array](assets/2015-04-27-161853_1266x829_scrot.png)](https://blog.algolia.com
/wp-content/uploads/2015/04/2015-04-27-161853_1266x829_scrot.png) It's a
simple JavaScript file, [exporting test cases as an
array](https://github.com/algolia/algoliasearch-client-
js/blob/1c8df9d09d683915a414e7df87a14236e50dd53c/test/spec/common/index/test-
cases/addUserKey.js#L1)

Creating a testing stack that understands theses test-cases was some work. But
the reward was worth it: the TDD feedback loop is great. Adding a new feature
is easy: fire editor, add test, implement annnnnd done.

### Advanced tests

Complex test cases like JSONP fallback, timeouts and errors, were handled in
separate, [more advanced tests](https://github.com/algolia/algoliasearch-
client-js/blob/1c8df9d09d683915a414e7df87a14236e50dd53c/test/spec/browser
/request-strategy/use-JSONP-when-XHR-error.js):

[![Our testing
stack rely on substack/tape](assets/2015-04-27-162229_1258x982_scrot.png)](https://blog.algolia.com/wp-content/uploads/2015/04/2015-04-27-162229_1258x982_scrot.png) Here we test
that we are using JSONP when XHR fails

## Testing workflow

To be able to run our tests we chose
[defunctzombie/zuul](https://github.com/defunctzombie/zuul/).

### Local development

For local development, we have an **npm test** task that will:

  * launch the browser tests using [phantomjs](http://phantomjs.org/),
  * run the Node.js tests,
  * lint using [eslint](http://eslint.org/).

You can [see the task](https://github.com/algolia/algoliasearch-client-
js/blob/1c8df9d09d683915a414e7df87a14236e50dd53c/package.json#L12) in the
package.json. Once run it looks like this:

[![640 passing
assertions and counting!](assets/2015-04-27-170339_976x845_scrot.png)](https://blog.algolia.com/wp-content/uploads/2015/04/2015-04-27-170339_976x845_scrot.png) 640 passing
assertions and counting!

But phantomjs is no real browser so it should not be the only answer to "Is my
module working in browsers?". To solve this, we have an [npm run
dev](https://github.com/algolia/algoliasearch-client-
js/blob/1c8df9d09d683915a414e7df87a14236e50dd53c/package.json#L16) task that
will expose our tests in a simple web server accessible by any browser:

[![All of theses
features are provided by defunctzombie/zuul](assets/2015-04-27-170757_1288x486_scrot.png)](https://blog.algolia.com/wp-content/uploads/2015/04/2015-04-27-170757_1288x486_scrot.png) All of theses
features are provided by defunctzombie/zuul

Finally, if you have [virtual machines](https://www.virtualbox.org/), you can
test in any browser you want, all locally:

[![Here's a
VirtualBox setup created with xdissent/ievms](assets/2015-04-27-171034_1296x385_scrot.png)](https://blog.algolia.com/wp-content/uploads/2015/04/2015-04-27-171034_1296x385_scrot.png) Here's a
VirtualBox setup created with
[xdissent/ievms](https://github.com/xdissent/ievms)

What comes next after setting up a good local development workflow? Continuous
integration setup!

### Continuous integration

[defunctzombie/zuul](https://github.com/defunctzombie/zuul/) supports running
tests using [Saucelabs](https://saucelabs.com/) browsers. Saucelabs provides
browsers as a service (manual testing or Selenium automation). It also has a
nice OSS plan called [Opensauce](https://saucelabs.com/opensauce/). We patched
our [.zuul.yml](https://github.com/algolia/algoliasearch-client-
js/blob/1c8df9d09d683915a414e7df87a14236e50dd53c/.zuul.yml) configuration file
to specify what browsers we want to test. You can find all the details in
[zuul's wiki](https://github.com/defunctzombie/zuul/wiki/Cloud-testing).

Now there's only one missing piece: [Travis CI](https://travis-ci.org/).
Travis runs our tests in all browsers defined in our .zuul.yml file. Our
[travis.yml](https://github.com/algolia/algoliasearch-client-
js/blob/1c8df9d09d683915a414e7df87a14236e50dd53c/.travis.yml) looks like this:

[![All platforms are tested using travis
matrixes](assets/2015-04-27-172249_1248x868_scrot.png)
](https://blog.algolia.com/wp-content/uploads/2015/04/2015-04-27-172249_1248x868_scrot.png)
All platforms are tested using a [travis build
matrix](http://docs.travis-ci.com/user/build-configuration/#The-Build-Matrix)

Right now tests are taking a bit too long so we will soon split them between
desktop and mobile.

We also want to to tests on pull requests using only latest stable versions of
all browsers. So that it does not takes too long. As a reward, we get a [nice
badge](https://docs.saucelabs.com/reference/status-images/) to display in our
Github [readme](https://github.com/algolia/algoliasearch-client-js#algolia-
search-api-client-for-javascript):

[![Gray color
means the test is currently
running](assets/2015-04-27-174139_920x306_scrot.png)](https://blog.algolia.com
/wp-content/uploads/2015/04/2015-04-27-174139_920x306_scrot.png) Gray color
means the test is currently running

# Challenge #2: redesign and rewrite

Once we had a usable testing stack, we started our rewrite, the [V3
milestone](https://github.com/algolia/algoliasearch-client-
js/issues?q=milestone%3AV3) on Github.

## Initialization

We dropped the **new AlgoliaSearch()** usage in favor of just
**algoliasearch()**. It allows us to hide implementation details to our API
users.

Before:

> new AlgoliaSearch(applicationID, apiKey, opts);

After:

> algoliasearch(applicationID, apiKey, opts);

## Callback convention

Our JavaScript client now follows the [error-
first](http://fredkschott.com/post/2014/03/understanding-error-first-
callbacks-in-node-js/) and [callback-last](http://blog.izs.me/post/59142742143
/designing-apis-for-asynchrony) conventions. We had to break some methods to
do so.

Before:

> client.method(param, callback, param, param);

After:

> client.method(params, param, param, params, callback);

This allows our callback lovers to use libraries like
[caolan/async](https://github.com/caolan/async) very easily.

## Promises and callbacks support

Promises are a great way to handle the asynchronous flow of your application.

> Promise partisan? Callback connoisseur? My API now lets you switch between
the two! [http://t.co/uPhej2yAwF](http://t.co/uPhej2yAwF) (thanks
[@NickColley](https://twitter.com/NickColley)!)

>

> — pouchdb (@pouchdb) [March 10,
2015](https://twitter.com/pouchdb/status/575310959947956224)

We implemented both promises and callbacks, it was nearly a no-brainer. In
every command, if you do not provide a callback, you get a [Promise](https://d
eveloper.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise).
We use native promises in [compatible
environments](http://caniuse.com/#feat=promises) and
[jakearchibald/es6-promise](https://github.com/jakearchibald/es6-promise/) as
a polyfill.

## AlgoliaSearchHelper removal

The main library was also previously exporting window.AlgoliaSearchHelper to
ease the development of awesome search UIs. We externalized this project and
it now has now has a new home at [algolia/algoliasearch-helper-
js](https://github.com/algolia/algoliasearch-helper-js).

## UMD

> UMD: JavaScript modules that run anywhere

The previous version was directly exporting multiple properties in the
**window** object. As we wanted our new library to be easily compatible with a
broad range of module loaders, we made it [UMD](https://github.com/umdjs/umd)
compatible. It means our library can be used:

  * with a simple `<script>`, it will export **algoliasearch** in the **window** object
  * using [browserify](http://browserify.org/), [webpack](http://webpack.github.io/), [requirejs](http://requirejs.org/): any module loader
  * in [Node.js](https://nodejs.org/)

This was achieved by writing our code in a
[CommonJS](http://en.wikipedia.org/wiki/CommonJS) style and then use the
standalone build feature of browserify.

[![see browserify
usage](assets/2015-04-28-224616_1126x129_scrot.png)](https://blog.algolia.com/wp-content/uploads/2015/04/2015-04-28-224616_1126x129_scrot.png) see
[browserify usage](https://github.com/substack/node-
browserify#usage)

## Multiple builds

Our JavaScript client isn't only one build, we have multiple builds:

  * vanilla JavaScript using native xhrs and Promises
  * jQuery build using [jQuery.ajax](http://api.jquery.com/jquery.ajax/) and returns [jQuery promises](https://api.jquery.com/category/deferred-object/)
  * AngularJS build is using [$http](https://docs.angularjs.org/api/ng/service/$http) service and returns [AngularJS promises](https://docs.angularjs.org/api/ng/service/$q)
  * [Parse.com](https://parse.com/) build is using parse cloud [http](https://parse.com/docs/cloud_code_guide#networking) and [promises](https://parse.com/docs/js_guide#promises)
  * Node.js, iojs are using the [http module](https://nodejs.org/api/http.html) and native Promises

Previously this was all handled in the main JavaScript file, leading to unsafe
[code like this](https://github.com/algolia/algoliasearch-client-js/blob/ba71a
68005945c73f7157a8222a9c758e332eceb/src/algoliasearch.js#L541-L553):

[![2015-04-28-225743_1629x497_scrot](assets/2015-04-28-225743_1629x497_scrot.png)](https://blog.algolia.com/wp-content/uploads/2015/04/2015-04-28-225743_1629x497_scrot.png)

How do we solve this? Using inheritance! JavaScript prototypal inheritance is
the new code smell in 2015. For us it was a good way to share most of the code
between our builds. As a result every entry point of our builds are inheriting
from the [src/AlgoliaSearch.js](https://github.com/algolia/algoliasearch-
client-js/blob/1c8df9d09d683915a414e7df87a14236e50dd53c/src/AlgoliaSearch.js).

Every build then need to define how to:

  * do http request through [AlgoliaSearch.prototype._request](https://github.com/algolia/algoliasearch-client-js/blob/1c8df9d09d683915a414e7df87a14236e50dd53c/src/browser/builds/algoliasearch.js#L46-L150)
  * return promises with [AlgoliaSearch.prototype._promise](https://github.com/algolia/algoliasearch-client-js/blob/1c8df9d09d683915a414e7df87a14236e50dd53c/src/browser/builds/algoliasearch.js#L167-L179)
  * use a request fallback where needed with [AlgoliaSearch.prototype._request.fallback](https://github.com/algolia/algoliasearch-client-js/blob/1c8df9d09d683915a414e7df87a14236e50dd53c/src/browser/builds/algoliasearch.js#L152-L165)

Using a simple inheritance pattern we were able to solve a great challenge.

[![Example of the
vanilla JavaScript
build](assets/2015-04-28-232019_1153x526_scrot.png)](https://blog.algolia.com
/wp-content/uploads/2015/04/2015-04-28-232019_1153x526_scrot.png) Example of
the vanilla JavaScript build

Finally, we have [a build script](https://github.com/algolia/algoliasearch-
client-js/blob/1c8df9d09d683915a414e7df87a14236e50dd53c/scripts/build) that
will generate all the needed files for each environment.

## Challenge #3: backward compatibility

We could not completely modernize our JavaScript clients while keeping a full
backward compatibility between versions. We had to break some of the previous
usages to level up our JavaScript stack.

But we also wanted to provide a good experience for our previous users when
they wanted to upgrade:

  * we re-exported previous constructors like window.AlgoliaSearch*. But we now **throw** if it's used
  * we wrote a clear [migration guide](https://github.com/algolia/algoliasearch-client-js/wiki/Migration-guide-from-2.x.x-to-3.x.x) for our existing Node.js and JavaScript users
  * we used [npm deprecate](https://docs.npmjs.com/cli/deprecate) on our previous Node.js module to inform our current user base that we moved to a new client
  * we created legacy branches so that we can continue to push critical updates to previous versions when needed

## Make it isomorphic!

Our final step was to make our JavaScript client work in both Node.js and the
browser.

Having separated the builds implementation helped us a lot, because the
Node.js build is a regular build only using the http module from Node.js.

Then we only had to tell module loaders to load **index.js** on the server and
**src/browser/..** in browsers.

This last step was done by configuring browserify in [our
package.json](https://github.com/algolia/algoliasearch-client-
js/blob/1c8df9d09d683915a414e7df87a14236e50dd53c/package.json#L5-L8):

[![the browser field from browserify also works in webpack](assets/2015-04-28-232444_1275x569_scrot.png)](https://blog.algolia.com/wp-content/uploads/2015/04/2015-04-28-232444_1275x569_scrot.png) the [browser
field](https://github.com/substack/node-browserify#browser-field) from
browserify also works in webpack

If you are using the **algoliasearch** module with browserify or webpack, you
will get our browser implementation automatically.

The **faux-jax** library is released under MIT like all our open source
projects. Any feedback or improvement idea are welcome, we are dedicated to
make our JS client your best friend :)

