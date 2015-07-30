<!--
{
"name" : "content-management-social",
"version" : "0.1",
"title" : "Content Management Social",
"description" : "TBD.",
"freshnessDate" : 2015-07-30,
"homepage" : "http://docs.basho.com/riak/latest/dev/data-modeling/",
"canonicalSource" : "http://docs.basho.com/riak/latest/dev/data-modeling/",
"license" : "All Rights Reserved"
}
-->

<!-- @section -->

## User Accounts

User accounts tend to rely on fairly straightforward data models. One way of storing user account data in Riak would be store each user's data as a JSON object in a bucket called `users` (or whatever you wish). Keys for user data objects could be constructed using application-specific considerations. If your application involves user logins, for example, the simplest and most read-efficient strategy would be to use the login username as the object key. The username could be extracted upon login, and a read request could be performed on the corresponding key.

There are, however, several drawbacks to this approach. What happens if a user wants to change their username later on? The most common solution would be to use a UUID-type key for the user and store the user's username as a [secondary index](http://docs.basho.com/riak/latest/dev/using/2i/) for efficient lookup.

### User Accounts Complex Case

For simple retrieval of a specific account, a user ID (plus perhaps a secondary index on a username or email) is enough. If you foresee the need to make queries on additional user attributes (e.g. creation time, user type, or region), plan ahead and either set up additional secondary indexes or consider using [Riak Search](http://docs.basho.com/riak/latest/dev/using/search/) to index the JSON contents of the user account.

### User Accounts Community Examples

<!-- @asset, "contentType": "outlearn/video", "provider": "vimeo", "url": "https://player.vimeo.com/video/47535803" -->

Ben Mills, a developer at Braintree, discusses how their backend team came to find and begin to integrate Riak into their production environment. They also cover their model and repository framework for Ruby, Curator. Check out more details and slides on the [Riak blog.](http://basho.com/blog/technical/2012/08/14/riak-at-braintree/)

<!-- @section -->

## User Settings and Preferences

For user account-related data that is simple and frequently read but rarely changed (such as a privacy setting or theme preference), consider storing it in the user object itself. Another common pattern is to create a companion user settings-type of object, with keys based on user ID for easy one-read retrieval.

### User Settings and Preferences Complex Case

If you find your application frequently writing to the user account or have dynamically growing user-related data such as bookmarks, subscriptions, or multiple notifications, then a more advanced data model may be called for.

<!-- @section -->

## User Events and Timelines

Sometimes you may want to do more complex or specific kinds of modeling user data. A common example would be storing data for assembling a social network timeline. To create a user timeline, you could use a `timeline` bucket in Riak and form keys on the basis of a unique user ID. You would store timeline information as the value, e.g. a list of status update IDs which could then be used to retrieve the full information from another bucket, or perhaps containing the full status update. If you want to store additional data, such as a timestamp, category or list of properties, you can turn the list into an array of hashes containing this additional information.

Note than in Riak you cannot append information to an object, so adding events in the timeline would necessarily involve reading the full object, modifying it, and writing back the new value.

### User Events and Timelines Community Examples

<!-- @asset, "contentType": "outlearn/video", "provider": "vimeo", "url": "https://player.vimeo.com/video/21598799" -->

This video was recorded at the March 2012 San Francisco Riak Meetup and is worth every minute of your time. Coda Hale and Ryan Kennedy of Yammer give an excellent and in depth look into how they built “Streamie”, user notifications, why Riak was the right choice, and the lessons learned in the process. Read more and get the slides in the Riak blog [here.](http://basho.com/blog/technical/2011/03/28/Riak-and-Scala-at-Yammer/)

<!-- @asset, "contentType": "outlearn/video", "provider": "vimeo", "url": "https://player.vimeo.com/video/44498491" -->

The team at Voxer has long relied on Riak as their primary data store for various production services. They have put Riak through its paces and have served as one of our more exciting customers and use cases: Riak was in place when they shot to the top of the App Store at the end of 2011\. We also love them because they open-sourced their Node.js client. Read more and get the slides in the Riak blog [here.](http://basho.com/blog/technical/2012/06/27/Riak-at-Voxer/)

<!-- @section -->

## Articles, Blog Posts, and Other Content

The simplest way to model blog posts, articles, or similar content is to use a bucket in Riak with some unique attribute for logical division of content, such as `blogs` or `articles`. Keys could be constructed out of unique identifiers for posts, perhaps the title of each article, a combination of the title and data/time, an integer that can be used as part of a URL string, etc.

In Riak, you can store content of any kind, from HTML files to plain text to JSON or XML or another document type entirely. Keep in mind that data in Riak is opaque, with the exception of [Riak Data Types](http://docs.basho.com/riak/latest/theory/concepts/crdts/), and so Riak won't “know” about the object unless it is indexed [using Riak Search](http://docs.basho.com/riak/latest/dev/using/search/) or [using secondary indexes](http://docs.basho.com/riak/latest/dev/using/2i/).

### Articles et al Complex Case

Setting up a data model for content becomes more complex based on the querying and search requirements of your application. For example, you may have different kinds of content that you want to generate in a view, e.g. not just a post but also comments, user profile information, etc.

For many Riak developers, it will make sense to divide content into different buckets, e.g. a bucket for comments that would be stored in the Riak cluster along with the posts bucket. Comments for a given post could be stored as a document with the same key as the content post, though with a different bucket/key combination. Another possibility would be to store each comment with its own ID. Loading the full view with comments would require your application to call from the posts and comments buckets to assemble the view.

Other possible cases may involve performing operations on content beyond key/value pairs. [Riak Search](http://docs.basho.com/riak/latest/dev/using/search/) is recommended for use cases involving full-text search. For lighter-weight querying, [using secondary indexes](http://docs.basho.com/riak/latest/dev/using/2i/) (2i) enables you to add metadata to objects to either query for exact matches or to perform range queries. 2i also enables you to tag posts with dates, timestamps, topic areas, or other pieces of information useful for later retrieval.

### Articles et al Community Examples

![milking-perf-from-riak](http://docs.basho.com/shared/2.1.1/images/milking-perf-from-riak.png)

Clipboard on [storing and searching data in Riak.](http://blog.clipboard.com/2012/03/18/0-Milking-Performance-From-Riak-Search)

![linkfluence-case-study](http://docs.basho.com/shared/2.1.1/images/linkfluence-case-study.png)

Linkfluence case study on using Riak to [store social web content](http://media.basho.com/pdf/Linkfluence-Case-Study-v2-1.pdf).

![ideeli-case-study](http://docs.basho.com/shared/2.1.1/images/ideeli-case-study.png)

ideeli case study on [serving web pages with Riak](http://basho.com/assets/Basho-Case-Study-ideeli.pdf).
