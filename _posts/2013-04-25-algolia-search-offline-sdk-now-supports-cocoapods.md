---
layout: post
title: Algolia Search Offline SDK now supports Cocoapods
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

We have great news for our iOS and OS X users: our Offline SDK is now
available as a _[CocoaPods][1] _dependency_._

Cocoapods is a popular dependency management tool that lets you specify the
libraries (dependencies) you want to use for your project in an easy-to-edit
text file (Podfile). It then fetches all the required libraries and sets up
your Xcode workspace.

You can now set up Algolia Search Offline in your iOS project with this line
in your Podfile:

`pod 'AlgoliaSearchOffline-iOS-SDK'`

You can also set up Algolia Search Offline in your OS X project with this
line:

`pod 'AlgoliaSearchOffline-OSX-SDK'`

Once you're done, simply use the "pod install" command to set up Algolia
Search Offline in your project. Now it's easy to manage library dependencies
for iOS and OS X projects!


[1]: http://www.cocoapods.org/
