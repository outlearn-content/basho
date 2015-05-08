

Retrieves information about a Riak Search index.


<!-- @section -->

## Request

```
GET /search/index/<index_name>
```


<!-- @section -->

## Normal Response Codes

* `200 OK`


<!-- @section -->

## Typical Error Codes

* `404 Object Not Found` --- No Search index with that name is currently
    available
* `503 Service Unavailable` --- The request timed out internally


<!-- @section -->

## Response

If the index is found, Riak will output a JSON object describing the
index, including its name, the `n_val` associated with it, and the search
schema used by the index. Here is an example:

```json
{
  "name": "my_index",
  "n_val": 3,
  "schema": "_yz_default"
}
```
