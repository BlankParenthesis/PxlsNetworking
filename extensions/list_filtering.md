List Filtering
==============
Implementing this extension allows any defined list endpoints to be filtered using query parameters.

For any list endpoint returning objects of type X, query parameters may be specified with keys from the keys on X.
The value of each query key specifies that only objects whose field - with a name matching the key - has the same value are returned.
For example, to get all Placement objects with x equal to 12 and y equal to 57, the client could perform a GET request to `/board/pixels?x=12&y=57`.
Similarly, to get all Placements of the color 0, the client could perform a GET request to `/board/pixels?color=0`.

Some fields are too specific to warrant filtering by exact value, such as Timestamps.
To allow for filtering without knowing a specific value, numeric values can be specified as a range.
This is done by joining the start number followed by the end number in a range with two period characters.
For example, to search for all Placements that occurred within the first minute of the Unix epoch, the client could perform a GET request to `/board/pixels?modified=0..60`.
Ranges include both the start and end numbers.

--------------------------------------------------------------------------------

## /info
### GET
#### Response
```typescript
{
	"extensions": ["list_filtering"];
}
```