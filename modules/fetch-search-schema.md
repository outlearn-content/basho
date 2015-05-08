

Retrieves a Riak search schema.


<!-- @section -->

## Request

```
GET /search/schema/<schema_name>
```


<!-- @section -->

## Normal Response Codes

* `200 OK`


<!-- @section -->

## Typical Error Codes

* `404 Object Not Found`
* `503 Service Unavailable` --- The request timed out internally


<!-- @section -->

## Response

If the schema is found, Riak will return the contents of the schema as
XML (all Riak Search schemas are XML).
