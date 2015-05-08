Riak has a rich, full-featured HTTP 1.1 API. This is an overview of the
operations you can perform via HTTP and can be used as a guide for
developing a compliant client. All URLs assume the default configuration
values where applicable. All examples use `curl` to interact with Riak.

> **URL Escaping**
Buckets, keys, and link specifications may not contain unescaped
slashes. Use a URL-escaping library or replace slashes with `%2F`.
