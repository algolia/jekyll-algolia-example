---
layout: post
title: Introducing Easier Onboarding and Activation with Connectors
author:
  login: sylvain
  email: sylvain@algolia.com
  display_name: Sylvain Utard
  first_name: Sylvain
  last_name: Utard
---

Most of our users are **technical**. They love writing code, and we love
providing API clients in the major programming languages to them (we are
currently [supporting 10 platforms][1].

They are doers. They love prototyping. Just like us, they work for startups
which need to move fast, and get things done, keeping in mind that done is
better than perfect. It is very important that they **don't want to waste
time**. In this post, I will explain how one would have used our API up to
now, and how we introduced SQL and MongoDB connectors for easier onboarding,
integration and testing.

## Before: The first steps with our API

Up until now, our onboarding process asked you to try the API by uploading
your data. We emphasized our [documentation][2], and
we made sure our users would not need more than a few minutes to integrate our
[REST API][3]. Nevertheless, exporting your
application's data to a JSON or CSV file is often more complex than it
appears, especially when you have millions of rows - and especially because
developers are lazy :) No worries, that's [totally
OK][4]. It is something you may not be willing to do, especially
just to try a service, so we decided to try something else.

### Initial import

90% of our users are using a SQL or MongoDB database. Exporting a table or a
collection to a JSON file can be easy if you're using a framework, for example
Ruby on Rails:

```ruby
File.open("/tmp/export.json", "w") do |f|
  f << MyActiveRecordModel.all.to_json
end
```

...or more annoying, for example when using PHP without any framework:

```php
mysql_connect('localhost', 'mysql_user', 'mysql_password');
mysql_set_charset('utf8');
$results = array();
$q = mysql_query("SELECT * FROM YourTable");
if ($q) {
  while (($row = mysql_fetch_assoc($q))) {
    array_push($results, $row);
  }
}
$fp = fopen('/tmp/export.json', 'w');
fwrite($fp, json_encode($results));
fclose($fp);
```

Anyway, in both cases it gets harder if you want to export millions of rows
without consuming hundreds GB of RAM. So you will need to use our API clients:

    
```ruby
index = Algolia::Index.new "YourIndex"
MyActiveRecordModel.find_in_batches(1000) do |objects|
  index.add_objects(objects)
end
# that's actually what `MyActiveRecordModel.reindex!` does

mysql_connect('localhost', 'mysql_user', 'mysql_password');
mysql_set_charset('utf8');
$limit = 1000;
$start = 0;
$index = $client->initIndex('YourIndexName');
while (true) {
  $q = mysql_query("SELECT * FROM YourTable LIMIT " . $start . "," . $limit);
  $n = 0;
  if ($q) {
    $objects = array();
    while(($row = mysql_fetch_assoc($q))) {
      array_push($objects, $row);
      ++$n;
    }
    $index->addObjects($objects);
  }
  if ($n != $limit) {
    break;
  }
  $start += $n;
}
```

### Incremental updates

Once imported, you will need to go further and keep your DB and our indexes
up-to-date. You can either:

  * Clear your index and re-import all your records hourly/daily with the previous methods: 
    * non-intrusive,
    * not real-time,
    * not durable,
    * need to import your data to a temporary index + replace the original one atomically once imported if you want to keep your service running while re-importing

Or

  * Patch your application/website code to replicate every add/delete/update operations to our API: 
    * real-time,
    * consistent & durable,
    * a little intrusive to some people, even though it is only a few lines of code ([see our documentation][5]

## After: Introducing connectors

Even if we did recommend you to modify your application code to replicate all
add/delete/update operations from your DB to our API, this should not be the
only option, especially to test Algolia. Users want to be convinced before
modifying anything in their production-ready application/website. This is why
we are really proud to release 2 open-source connectors: a non-intrusive and
efficient way to synchronize your current SQL or MongoDB database with our
servers.

### SQL connector

Github project: [algolia/jdbc-java-connector][6] (MIT license, we love pull-requests :))

The connector starts by enumerating the table and push all matching rows to
our server. If you store the last modification date of a row in a field, you
can use it in order to send all detected updates every 10 seconds. Every 5
minutes, the connector synchronizes your database with the index by adding the
new rows and removing the deleted ones.

    
```java
jdbc-connector.sh --source "jdbc:mysql://localhost/YourDB"  
  --username mysqlUser --password mysqlPassword             
  --selectQuery "SELECT * FROM YourTable" --primaryField id 
  --updateQuery "SELECT * FROM YourTable WHERE updated_at > _$"
  --updatedAtField updated_at 
  --applicationId YourApplicationId --apiKey YourApiKey --index YourIndexName
  ```

If you don't have an updated_at  field, you can use:

    
```java
jdbc-connector.sh --source "jdbc:mysql://localhost/YourDB"  
  --username mysqlUser --password mysqlPassword             
  --selectQuery "SELECT * FROM YourTable" --primaryField id 
  --applicationId YourApplicationId --apiKey YourApiKey --index YourIndexName
```

The full list of features is available on [Github][7] (remember, we ♥ feature and pull-requests)!

### MongoDB connector

Github
project: [algolia/mongo-connector][8]

This connector has been forked from [10gen-lab's official
connector][9] and is based on
MongoDB's [operation logs][10]. This means you will need to start your mongod  server specifying a
[replica set][11].
Basically, you need to start your server with: mongod --replSet
REPLICA_SET_IDENTIFIER. Once started, the connector will replicate each
addition/deletion/update to our server, sending a batch of operations every 10
seconds.

    
    mongo-connector -m localhost:27017 -n myDb.myCollection 
      -d ./doc_managers/algolia_doc_manager.py              
      -t YourApplicationID:YourApiKey:YourIndex

The full features list is available on [Github][12] (we ♥ feature and pull-requests).

## Conclusion: Easier Onboarding, Larger Audience!

Helping our users to onboard and try Algolia without writing a single line of
code is not only a way to attract more non-technical users; It is also a way
to save the time of our technical but overbooked users, allowing them to be
convinced without wasting their time before really implementing it.

Those connectors are open-source and we will continue to improve them based on
your feedback. Your feature requests are welcome!


[1]: http://www.algolia.com/doc/apiclients)
[2]: http://www.algolia.com/doc
[3]: http://www.algolia.com/doc/rest
[4]: http://www.codinghorror.com/blog/2005/08/how-to-be-lazy-dumb-and-successful.html
[5]: http://www.algolia.com/doc)
[6]: https://github.com/algolia/jdbc-java-connector
[7]: https://github.com/algolia/jdbc-java-connector
[8]: https://github.com/algolia/mongo-connector
[9]: https://github.com/10gen-labs/mongo-connector
[10]: http://docs.mongodb.org/manual/core/replica-set-oplog/
[11]: http://docs.mongodb.org/manual/tutorial/deploy-replica-set/
[12]: https://github.com/algolia/mongo-connector
