<!--
{
"name" : "high-read-write",
"version" : "0.1",
"title" : "Riak Use Cases: High Read/Write, Simple Applications",
"description" : "TBD.",
"freshnessDate" : 2015-07-30,
"homepage" :   "http://docs.basho.com/riak/latest/dev/data-modeling/",
"canonicalSource" : "http://docs.basho.com/riak/latest/dev/data-modeling/",
"license" : "All Rights Reserved"
}
-->
<!-- @section -->
# Overview

The following are examples of Riak use cases that require high read/write performance without necessarily utilizing complex data structures.

<!-- @section -->

# Session Storage

Riak was originally created to serve as a highly scalable session store. This is an ideal use case for Riak, which is always most performant and predictable when used as a key/value store. Since user and session IDs are usually stored in cookies or otherwise known at lookup time, Riak is able to serve these requests with predictably low latency. Riak's content-type agnosticism also imposes no restrictions on the value, so session data can be encoded in many ways and can evolve without administrative changes to schemas.

<!-- @multipleChoice -->

In an ideal Riak use case

- [X] Keys are known at lookup time
- [ ] Schemas are clearly defined beforehand
- [ ] Latency is variable but generally low

Read again the previous paragraph to refresh your memory.

<!-- @end -->

### Complex Session Storage Case

Riak has features that allow for more complex session storage use cases. The [Bitcask](http://docs.basho.com/riak/latest/ops/advanced/backends/bitcask/) storage backend, for example, supports automatic expiry of keys, which frees application developers from implementing manual session expiry. Riak's [MapReduce](http://docs.basho.com/riak/latest/dev/using/mapreduce/) system can also be used to perform batch processing analysis on large bodies of session data, for example to compute the average number of active users. If sessions must be retrieved using multiple keys (e.g. a UUID or email address), [using secondary indexes](http://docs.basho.com/riak/latest/dev/using/2i/) can provide an easy solution.

### Session Storage Community Examples

<!-- @resource, "url": "https://player.vimeo.com/video/42744689"  -->

In this talk, recorded at the May 2012 San Francisco Riak Meetup, Armon Dadgar and Mitchell Hashimoto of Kiip give an overview of how and why they are using Riak in production, and the road they took to get there. One of the first subsystems they switched over to Riak was Sessions. You can also read the blog post and catch the slides [here.](http://basho.com/blog/technical/2012/05/25/Scaling-Riak-At-Kiip/)

 <!-- @openResponse, "text" : "Watch the video \"Scaling Riak at Kiip\" and write a short summary of those points in the video that matter most to you. Submit your summary here."-->

<!-- @section -->

## Serving Advertisements

Riak is often a good choice for serving advertising content to many different web and mobile users simultaneously with low latency. Content of this sort, e.g. images or text, can be stored in Riak using unique generated either by the application or by Riak. Keys can be created based on, for example, a campaign or company ID for easy retrieval.

### Serving Advertisements Complex Case

In the advertising industry, being able to serve ads quickly to many users and platforms is often the most important factor in selecting and tuning a database. Riak's tunable [Replication Properties](http://docs.basho.com/riak/latest/dev/advanced/replication-properties/) can be set to favor fast read performance. By setting R to 1, only one of N replicas will need to be returned to complete a read operation, yielding lower read latency than an R value equal to the number of replicas (i.e. R=N). This is ideal for advertising traffic, which primarily involves serving reads.

### Serving Advertisements Community Examples

<!-- @resource, "url": "https://player.vimeo.com/video/49775483" -->

Los Angeles-based OpenX will serves trillions of ads a year. In this talk, Anthony Molinaro, Engineer at OpenX, goes in depth on their architecture, how they’ve built their system, and why/how they’re switching to Riak for data storage after using databases like CouchDB and Cassandra in production.

 <!-- @openResponse, "text" : "Watch the video \"Riak at OpenX\" and write a short summary of those points in the video that matter most to you. Submit your summary here."-->

<!-- @section -->

## Log Data

A common use case for Riak is storing large amounts of log data, either for analysis [using MapReduce](http://docs.basho.com/riak/latest/dev/using/mapreduce/) or as a storage system used in conjunction with a secondary analytics cluster used to perform more advanced analytics tasks. To store log data, you can use a bucket called `logs` (just to give an example) and use a unique value, such as a date, for the key. Log files would then be the values associated with each unique key.

For storing log data from different systems, you could use unique buckets for each system (e.g. `system1_log_data`, `system2_log_data`, etc.) and write associated logs to the corresponding buckets. To analyze that data, you could use Riak's MapReduce system for aggregation tasks, such as summing the counts of records for a date or Riak Search for a more robust, text-based queries.

### Log Data Complex Case

For storing a large amount of log data that is frequently written to Riak, some users might consider doing primary storage of logs in a Riak cluster and then replicating data to a secondary cluster to run heavy analytics jobs, either over another Riak cluster or another solution such as Hadoop. Because the access patterns of reading and writing data to Riak is very different from the access pattern of something like a MapReduce job, which iterates over many keys, separating the write workload from the analytics workload will let you maintain higher performance and yield more predictable latency.

### Log Data Community Examples

![simon-analyzing-logs)](http://docs.basho.com/shared/2.1.1/images/simon-analyzing-logs.png)

Simon Buckle on [analyzing Apache logs with Riak.](http://www.simonbuckle.com/2011/08/27/analyzing-apache-logs-with-riak/)

 <!-- @openResponse, "text" : "Read the article about \"Analyzing Apache logs with Riak\" and write a short summary of those points in the article that matter most to you. Submit your summary here."-->

<!-- @section -->

## Sensor Data

Riak's scalable design makes it useful for data sets, like sensor data, that scale rapidly and are subject to heavy read/write loads. Many sensors collect and send data at a given interval. One way to model this in Riak is to use a bucket for each sensor device and use the time interval as a unique key (i.e. a date or combination of date and time), and then store update data as the value.

That data could then be queried on the basis of the interval. Alternatively, a timestamp could be attached to each object as a [secondary index](http://docs.basho.com/riak/latest/dev/using/2i/), which would allow you to perform queries on specific time interval ranges or to perform [MapReduce](http://docs.basho.com/riak/latest/dev/using/mapreduce/) queries against the indexes.

### Sensor Data Complex Case

If you are dealing with thousands or millions of sensors yet with very small data sets, storing all of a single device's updates as unique keys may be cumbersome when it comes to reading that device's data. Retrieving it all would mean calling a number of keys.

Instead, you could store all of a device's updates in a document with a unique key to identify the device. Stored as a JSON document, you could read and parse all of those updates on the client side. Riak, however, doesn't allow you to append data to a document without reading the object and writing it back to the key. This strategy would mean more simplicity and performance on the read side as a tradeoff for slightly more work at write time and on the client side.

It's also important to keep an eye out for the total size of documents as they grow, as we tend to recommend that Riak objects stay smaller than 1-2 MB and preferably below 100 KB. Otherwise, performance problems in the cluster are likely.
