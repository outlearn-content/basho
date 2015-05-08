

Deletes a Riak Search index.


<!-- @section -->

## Request

```
DELETE /search/index/<index_name>
```


<!-- @section -->

## Normal Response Codes

* `204 No Content` --- The index was successfully deleted (also returned
    if the index did not exist to begin with)


<!-- @section -->

## Typical Error Codes

* `503 Service Unavailable` --- The request timed out internally
