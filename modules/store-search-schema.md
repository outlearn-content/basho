

Creates a new Riak Search schema.


<!-- @section -->

## Request

```
PUT /search/index/<schema_name>
```


<!-- @section -->

## Required Form Data

In order to create a new Search schema, you must pass Riak a properly
formed XML schema. More information can be found in the Search
Schema document. If you've created a schema and stored it in the filed
`my_schema.xml` and would like to create a new schema called
`my_custom_schema`, you would use the following HTTP request:

```curl
curl -XPUT http://localhost:8098/search/schema/my_custom_schema \
  -H "Content-Type: application/xml" \
  --data-binary @my_schema.xml
```


<!-- @section -->

## Normal Response

* `204 No Content` --- The schema has been successfully created


<!-- @section -->

## Typical Error Codes

* `400 Bad Request` --- The schema cannot be created because there is
    something wrong with the schema itself, e.g. an XML formatting error
    that makes Riak Search unable to parse the schema
* `409 Conflict` --- The schema cannot be created because there is
    already a schema with that name
* `503 Service Unavailable` --- The request timed out internally
