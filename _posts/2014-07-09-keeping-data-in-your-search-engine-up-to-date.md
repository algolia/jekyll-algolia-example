---
layout: post
title: Keeping Data in your Search Engine Up-to-Date
author:
  login: julien
  email: julien.lemoine@algolia.com
  display_name: julien
  first_name: Julien
  last_name: Lemoine
---

When we developed the first version of Algolia Search, we put a lot of effort
into developing a data update API. It worked like this: You could send us a
modified version of your data as soon as the change appeared, even if it
concerned only a specific part of a record. For example, this batch of
information could be the updated price or number of reviews, and we would only
update this specific attribute in your index.

However, this initial plan did not take into account that most of our big
customers would not benefit from this API due to their existing
infrastructure. If you had not planned to catch all updates in your
architecture, or if you were not using a framework like Ruby on Rails, it
could be very difficult to even have a notification for any of these updates.
The solution in this case was to use a batch update on a regular basis. It was
a good method to use if you didn't want to change a single line of code in
your existing infrastructure, but the batch update was far from a cure-all.

## The problem of batch update

There are two main ways to perform a batch update on a regular basis:

  1. Scan your database and update all objects. This method is good if you have no delete operation, but if some data are removed from your database, you will need to perform an extra check to handle delete, which can be very slow.
  2. Clear the content of the index and import all your objects. With this method, you ensure that your index is well synchronized with your database. However, if you receive queries during the import, you will return partial results.  If interrupted, the whole rescan could break your relevance or your service.

So the two approaches are somewhat buggy and dangerous.

## Another approach: build a new index with another name

Since our API allows the creation of a new index with a different name, you
could have made your batch import in a new index. Afterward, you would just
need to update your front end to send queries to this new index.

Since all indexing jobs are done asynchronously, we first need to check that
an indexing job is finished. In order to do that, we return an integer (called
TaskID) that allows you to check if an update job is applied. Thus, you just
have to use the API to check that the job is indexed.

But then a problem arises with mobile applications: You cannot change the
index name of an application as easily, since most of the time, it is a
constant in the application code. And even for a website, it means that the
batch will need to inform your frontend that the index name is different. This
can be complex.

## The elegant solution: move operation

To solve these problems, we implemented a command that is well known on file
systems: **move**. You can move your new index on the old one, and this will
atomically update the content of the old index with the content of the new
one. With this new approach, you can solve all the previous update problems
with one simple procedure. Here's how you would update an index called
"MyIndex":

  1. Initialize an index "MyIndex.tmp"
  2. Scan your database and import all your data in "MyIndex.tmp"
  3. Move "MyIndex.tmp in "MyIndex"

You don't have to do any modification on your backend to catch modifications,
nor do you need to change the index name on the frontend. Even better, you
don't need to check the indexing status with our TaskID system since the
"move" operation will simply be queued after all "adds". All queries will go
to the new index when it is ready.

## The beauty of the move command

This command is so elegant that even customers who had been sending us
realtime updates via our updates API have decided to use this batch update on
a regular basis. The move command is a good way to ensure that there are no
bugs in your update code, nor divergence between your database and Algolia.

This operation is supported in our twelve API Clients. We go even further in
our Ruby on Rails integration: You need only use the 'reindex' command
(introduced in 1.10.5) to automatically build a new temporary index and move
it on top of the existing one.

The move command is an example of how we try to simplify the life of
developers. If you see any other way we can help you, let us know and we'll do
our best to remove your pain!

