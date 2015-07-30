<!--
{
"name" : "migrating-to-riak",
"version" : "0.1",
"title" : "Migrating from an SQL Database to Riak",
"description" : "TBD.",
"freshnessDate" : 2015-07-30,
"homepage" : "http://docs.basho.com/riak/latest/dev/data-modeling/sql-migration/",
"canonicalSource" : "http://docs.basho.com/riak/latest/dev/data-modeling/sql-migration/",
"license" : "All Rights Reserved"
}
-->

<!-- @section -->

# Overview

Relational databases are powerful and reliable technologies, but there
are many [use cases](http://docs.basho.com/riak/latest/dev/data-modeling/) for which Riak is a better fit, e.g. when data
availability is more important than SQL-style queryability or when
relational databases begin to run into scalability problems. You can
find out more in [Why Riak](http://docs.basho.com/riak/latest/theory/why-riak/). If you decide that Riak is a better fit,
this tutorial walks you through migrating from an SQL system to Riak.

>**Use cases warning**

>Because data models vary so widely, it is difficult if not impossible to
generalize across all potential paths from an SQL database to Riak. This
document is intended only to suggest one possible approach to SQL data
migration&mdash;an approach that may not work well with your use case.

<!-- @section -->

## Our Example

Let's say that we've been storing a series of blog posts in
[PostgreSQL](http://www.postgresql.org/), in a database called `blog`
and a table called `posts`. This table has the following schema:

```sql
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    author VARCHAR(30) NOT NULL,
    title VARCHAR(50) NOT NULL,
    body TEXT NOT NULL,
    created DATE NOT NULL
);
```

A typical post looks like this when queried:

```sql
SELECT * FROM posts WHERE id = 99;
```

The response:

```
 id |   author   |             title              |            body            |  created
----+------------+--------------------------------+----------------------------+------------
 99 | John Daily | Riak Development Anti-Patterns | Writing an application ... | 2014-01-07
```

Our basic conversion and storage approach will be the following:

1. Each row will be converted into a JSON object storing all of the
   fields except the `id` field.
2. The `id` field will be excluded from the JSON object stored in Riak.
   It will not act as each object's [key](http://docs.basho.com/riak/latest/theory/concepts/keys-and-values/#keys).
   Instead, the key for each object will be the post's title, up to 30
   characters, lowercased and with hyphens separating the words in the
   title. And so the example post shown above will have the key
   `riak-development-anti-patterns`.
3. All of the JSON objects produced from the `posts` table will be
   stored in a single Riak [bucket](http://docs.basho.com/riak/latest/theory/concepts/Buckets/) called `posts`.
4. The keys for our various objects will be stored in a [Riak set](http://docs.basho.com/riak/latest/dev/using/data-types/#sets) so that all stored objects can be queried at once  if need be.

<!-- @section -->

## Converting the Table to a List

In this tutorial, we'll store a table housing a series of blog posts in
Riak using a [Python](https://www.python.org/) script relying on
[psycopg2](http://initd.org/psycopg/docs/), a PostgreSQL driver for
Python.

Using the pysopg2 library, we can establish a connection to our database
(we'll call the database `blog_db`) and create a
[cursor](http://www.postgresql.org/docs/9.2/static/plpgsql-cursors.html)
object that will allow us to interact with the `posts` table using
traditional SQL commands:

```python
import psycopg2

connection = psycopg2.connection('dbname=blog_db')
cursor = connection.cursor()
```

With that cursor, we'll execute a `SELECT * FROM posts` query and then
fetch the information from the cursor using the `fetchall` function:

```python
cursor.execute('SELECT * FROM posts')
table = cursor.fetchall()
```

The `table` object consists of a Python list of tuples that looks
something like this:

```python
[(1, 'John Doe', 'Post 1 title', 'Post body ...', datetime.date(2014, 1, 1)),
 (2, 'Jane Doe', 'Post 2 title', 'Post body ...', datetime.date(2014, 1, 2)),
 # more posts in the list
]
```

As we can see, psycopg2 has automatically converted the `created` row
from a Postgres `DATE` data type into a Python datetime. We'll need to
convert that datetime to a string when we convert each row to JSON in
the next section.

<!-- @section -->

## Converting Rows to JSON Objects

In the section above, we saw that psycopg2 converted each row of our
`posts` table into a tuple with five elements (one for each column in
our table). Tuples aren't a terribly useful data type to store in Riak,
so we'll convert each row tuple into a Python
[dictionary](https://docs.python.org/2/tutorial/datastructures.html#dictionaries)
instead. The official [Riak Python client](https://github.com/basho/riak-python-client)
automatically converts Python dictionaries to JSON to store in Riak, so
once we have a list of dictionaries instead of tuples, we can store
those dictionaries directly in Riak.

Converting rows in an SQL table to dictionaries can be tricky because
rows can contain a wide variety of data types, each of which must be
converted into one of the data types [compatible with
JSON](http://en.wikipedia.org/wiki/JSON#Data_types.2C_syntax_and_example).
That conversion is fairly straightforward in our example, as the `name`,
`title`, and `body` columns are automatically converted into strings.

The one tricky part will be the `date` column. Fortunately, Python's
[datetime](https://docs.python.org/2/library/datetime.html) library
makes this fairly simple. We can use the `strftime` function to
convert the `date` column into a formatted string. We'll use a
month-day-year format, i.e. `%m-%d-%Y`.

```python
import datetime

def convert_row_to_dict(row):
	return {
		'author': row[1],
		'title': row[2],
		'body': row[3],
		'created': row[4].strftime('%m-%d-%Y')
	}
```

That will convert each row into a dictionary that looks like this:

```json
{
  'author': 'John Daily',
  'title': 'Riak Development Anti-Patterns',
  'body': 'Writing an application ...',
  'created': '01-07-2014'
}
```

<!-- @section -->

## Storing Row Objects

Now that we can convert each row into a Python dictionary, we can store
each row in Riak directly. Our `store_row_in_riak` function will do two
things:

1. It will construct a key out of each post's title, taking the first 30
   characters, lowercasing the whole string, and then replacing all
   spaces with a hyphen, i.e. `This is a blog post` will be transformed
   into `this-is-a-blog-post`.
2. Each row will be converted into a proper Riak object and stored.

Here's our function:

```python
bucket = client.bucket('posts')

def store_row_in_riak(row):
	key = row[2][0:29].lower().replace(' ', '-')
	obj = RiakObject(client, bucket, key)
	obj.content_type = 'application/json'
	obj.data = convert_row_to_dict(row)
	obj.store()
```

As stated above, we'll want to store all of the objects' keys in a
[Riak set](http://docs.basho.com/riak/latest/theory/concepts/crdts/#sets) to assist us in querying the objects in the
future. We'll modify the `store_row_in_riak` function above to add each
key to a set:

```python
from riak.datatypes import Set

objects_bucket = client.bucket('posts')
key_set = Set(client.bucket_type('sets').bucket('key_sets'), 'posts')

def store_row_in_riak(row):
	key = row[0]
	obj = RiakObject(client, bucket, key)
	obj.content_type = 'application/json'
	obj.data = convert_row_to_dict(row)
	obj.store()
```

Now we can write an iterator that stores all rows:

```python
# Using our "table" object from above:

for row in table:
	store_row_in_riak(row)
```

Once all of those objects have been stored in Riak, we can perform
normal key/value operations to fetch them one by one. Here's an example,
using curl and Riak's [HTTP API](http://docs.basho.com/riak/latest/dev/references/http/):

```curl
curl http://localhost:8098/buckets/posts/keys/99
```

That will return a JSON object containing one of the blog posts from our
original table:

```json
{
  "author": "John Daily",
  "title": "Riak Development Anti-Patterns",
  "body": "Writing an application ...",
  "created": "01-07-2014"
}
```

But we can also fetch all of those objects at once if need be.
Previously, we stored the keys for all of our objects in a
[Riak set](http://docs.basho.com/riak/latest/theory/concepts/crdts/#sets). We can write a function that fetches all
of the keys from that set and in turn all of the objects corresponding
to those keys:

```python
from riak.datatypes import Set

set_bucket = client.bucket_type('sets').bucket('key_sets')
posts_bucket = client.bucket('posts')

def fetch_all_objects(table_name):
	keys = Set(client, bucket, table_name)
	for key in keys:
		return posts_bucket.get(key)

fetch_all_objects('posts')
```

That will return the full list of Python dictionaries we stored earlier.

<!-- @section -->

## Enhanced Discoverability with Secondary Indexes

While storing all of the objects' keys in a Riak set is a good way to be
able to fetch all objects from the `posts` bucket if necessary, a much
more powerful querying technique would be to mark each object with
[secondary indexes](http://docs.basho.com/riak/latest/dev/using/2i/) that enable us to fetch
particular blog posts on the basis of some piece of information about
those posts.

Let's say that we stored all of our blog posts with keywords attached,
and that our original schema actually looked like this:

```sql
CREATE TABLE posts (
    id SERIAL PRIMARY KEY,
    author VARCHAR(30) NOT NULL,
    title VARCHAR(50) NOT NULL,
    body TEXT NOT NULL,
    created DATE NOT NULL,
    keywords TEXT[] NOT NULL
);
```

Here's an example insert into that table:

```sql
INSERT INTO posts (author, title, body, created, keywords) VALUES
	('Basho', 'Moving from MySQL to Riak', 'Traditional database architectures...',
	current_date, '{"mysql","riak","migration","rdbms"}');
```

What we can do now is add a binary secondary index for each of these
keywords to each post. Let's write a function to take a Riak object
and attach a binary secondary index for each keyword:

```python
def add_keyword_2i_to_object(obj, keywords):
	for keyword in keywords:
		obj.add_index('keywords_bin', keyword)
```

Then we can insert that function into the `store_row_in_riak` function
that we created above:

```python
bucket = client.bucket('posts')

def store_row_in_riak(row):
	obj = RiakObject(client, bucket, row[0])
	obj.content_type = 'application/json'
	obj.data = convert_row_to_dict(row)
	add_keyword_2i_to_object(obj, row[5])
	obj.store()
```

Now, we can fetch blog posts on the basis of their keywords:

```python
bucket = client.bucket('posts')

def fetch_posts_by_keyword(keyword):
	for key in bucket.get_index('keywords_bin', keyword):
		return bucket.get(key)
```

This will then return a list of objects marked with keyword metadata.
