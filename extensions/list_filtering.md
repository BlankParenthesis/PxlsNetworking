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

Ranges may be unbounded.
This is indicated by not specifying the start or end of the range.
For example, to search for all Placements that occurred after the first minute of the Unix epoch, the client could perform a GET request to `/board/pixels?modified=60..`.

Some objects have nested fields.
For example, Member objects from the [factions extension](./factions.md) have an approval field.
This approval field does not contain a simple value but an object with the following type:
```typescript
{
	"member": boolean;
	"faction": boolean;
}
```
These fields can still be filtered by their values.
An object fields inner fields can be accessed by joining the field path from outermost to innermost with a period character.
In order to get all members of a faction which have approval from the faction, the client could perform a GET request to `/factions/0/members?approval.faction=true`.

Some fields have an array value.
These fields may not be directly filtered, but may be filtered on their length.
Arrays can be considered as an object with the following type:
```typescript
{
	"length": number;
}
```
We can access the length of an array using what was previously defined regarding nested fields.
For example, in order to get all reports (see the [reports extension](./reports.md)) which have been modified more than twice, the client could perform a GET request to `/reports?history.length=2..`

--------------------------------------------------------------------------------

## /info
### GET
#### Response
```typescript
{
	"extensions": ["list_filtering"];
}
```