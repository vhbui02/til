# ETag

`ETag` is a request header.

- helps caches be more efficient and save bandwidth
- web server does not need to resend a full response if content has not changed
- prevent mid-air collisions

## How to create?

1. Hash of the content
2. Hash of the last modification timestamp
3. Revision number.

## Mid-air collisions

When editing a wiki content, the current wiki is hashed and put into `ETag` of a GET HTTP Response.
When saving changes to a wiki page, the POST HTTP Request will contain `If-Match` HTTP Request Header containing the previous ETag value to check freshness against.
=> If the hashes don't match, a `412 Precondition Failed` error is thrown.

## Caching of unchanged resources

User visits a given URL again that has an `ETag` set and it's deemed too old to be considered usable, the client sends the value of `If-None-Match` Request Header with `ETag` value.
The server compares the client's `ETag` from with `If-None-Match` and `ETag` for its current version of the resource. If matched, the server sends back a `304 Not Modified` status.
