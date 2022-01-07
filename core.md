Core Specification
==================
All timestamps are Unix time unless specified otherwise. 
They should be treated as the number type in practice but are represented clearly as "Timestamp" here.

A number of core protocol extensions exist. 
Within these extensions, endpoints declared either here or in other extensions may be re-declared.
These **duplicate response type definitions should be treated as the union of implemented definitions**.
This allows extensions to add new fields to responses in a flexible way.


To be as RESTful as possible, objects have a canonical address which can be used to refer to them.
Where an object is returned elsewhere, a Reference to it will be returned instead.
A Reference for an object T can be described with the following type:
```typescript
type Reference<T> = {
	"uri": string;
	"view"?: T;
};
```
where `uri` points to a location where `T` can be obtained with a simple GET request.
The `view` field may be populated by servers so that clients can operate faster and with less requests.

Clients may need to specify an object as part of a request.
By using the URI of the object, clients need not know anything about internal IDs used by the server.

Server implementations may choose to logically arrange their reference URIs (e.g. `/users/{user_id}`), but are not required to do so.
Clients should not assume such a structure, nor should they assume any server will popular the `view` field.


Some endpoints return a list of objects rather that a single one.
Sometimes, too many objects will exist in this list to be reasonably sent in a single request.
To solve this, all list endpoints return paginated responses.
For an object type X, a Paginated List of X can be defined by the following type:
```typescript
{
	"items": X[];
	"next"?: string;
	"previous"?: string;
}
```
Where `next` and `previous` are URIs pointing to the continuations in their respective directions of the list.
The presence of these fields should be used to determine if the request completes or starts a list.

In requesting a Paginated list, clients may specify an upper limit of responses in a single page with the `limit` query parameter.
Server implementations must not return more items than this number but may return less even if the list is not completed by the current page.


At the core of Pxls is the board on which pixels are placed.
This is represented here as the Board object defined by the following type:
```typescript
{
	"name": string;
	"created_at": Timestamp;
	"shape": number[][]
	"palette": Map<number, {
		"name": string;
		"value": number;
	}>;
	"max_pixels_available": number;
}
```
The `value` field of each entry in `palette` is a 4-byte ARGB color value.

`shape` indicates the ordering of data for the board.
It usually contains a single array of size 2 with its elements representing width and height respectively.
However, it may contain multiple arrays in which case each array indicates the dimensions of a virtual grid with each grid entry being the next item in the array.
This allows effective chunking of board contents using only range requests to board data endpoints.

As an example, a shape of `[[2, 2], [500, 500]]` represents a 1000 by 1000 area, broken into 4 subareas.
The first 250,000 pixels are entirely contained in the first 500 by 500 area.
A shape of `[[2, 2], [2, 2], [250, 250]]` would again have the same size and even the same first 250,000 pixels.
It would differ by orienting the first 62,500 pixels in the first 250 by 250 area rather than the first 500 by 125 area as the previous shape would.

All dimensions should be interpreted as being of the same length by padding any entries smaller than others with 1s at the end.

Clients must be allowed to make valid requests with a range size equal to the product of the final shape dimensions, aligned to the same size.
For example, for a shape of `[[8,8], [128, 128]]` a server must accept each of these requests: `Range: bytes=0-16384`, `Range: bytes=16384-32768`, `Range: bytes=-16384`, `Range: bytes=1032192-`. These examples are non-exhaustive, obviously the server must respond to many other valid ranges as well. Additionally, the server may respond to arbitrary range requests if it chooses.

The server should respond to such requests with a 206 partial content response with the bytes unit type as other unit types are prone to being dropped by proxy servers.

Servers need not support multipart ranges — the hassle of properly supporting the multipart/byteranges response type (the boundary in particular) makes this less desirable than it initially seems.

It should be noted that servers can reject requests for the entire board.
Such rejections should be done with a 416 response even if the original request did not contain a range.
All clients should support ranges to be compatible.
This is so that servers can reject blind client requests for chunked boards which are too large to reasonably process as one object, saving the client from itself.

For orientation purposes, the default board orientation is left→right, then top→bottom, then back→front.
Higher order boards are up to client interpretation.


Of equivalent importance to the board is the ability to modify it with Placements.
A Placement object represents a change of board state at a particular time and is defined by the following type:
```typescript
{
	"position": number;
	"color": number;
	"modified": Timestamp;
}
```
Placements are often not functional distinguishable from pixels on the board and the two concepts are mostly treated interchangeably.


Nearly all requests by the client can be rejected by the server implementation.
It is useful for the client to know before making such requests if it will be allowed to do so.
For this reason, each method of each endpoint is associated with a string denoting the permission to use that endpoint.
A list of these strings is sent to the client in the `/access` endpoint.
Some permissions are context sensitive.
These usually have "current" or "owned" somewhere in their string.
Such permissions should be used if the specified object belongs to the client user in some way and the client lacks the general permission.

--------------------------------------------------------------------------------

## /info
### GET
Gets information about the server implementation.
#### Response
```typescript
{
	"name"?: string;
	"version"?: string;
	"source"?: string;
	"extensions": string[];
}
```
Extension definitions redefine the `extensions` field in this request.
Implementations should return the union of all lists defined by every implemented extensions as a set.
#### Errors
| Response Code | Cause                      |
|---------------|----------------------------|
| 403 Forbidden | Missing permission `info`. |

*NOTE: Missing this permission effectively prevents the client from using extensions.*

--------------------------------------------------------------------------------

## /access
### GET
Gets information about what this client can do without encountering a permissions error.
#### Response
```typescript
{
	"permissions": string[];
}
```

The `permissions` field  contains a list of permissions strings.
It represents the actions the client can take without encountering a permissions error.

--------------------------------------------------------------------------------

## /boards
### GET
Lists all Board objects.
#### Response
A Paginated List of Board References.
#### Errors
| Response Code | Cause                            |
|---------------|----------------------------------|
| 403 Forbidden | Missing permission `boards.list` |

### POST
Creates a new Board object.
#### Request
```typescript
{
	"name": string;
	"shape": number[][]
	"palette": Map<number, {
		"name": string;
		"value": number;
	}>;
}
```
#### Response
A Board Reference.
##### Headers
| Header | Value                                |
|--------|--------------------------------------|
| Location | The location of the created Board. |
#### Errors
| Response Code | Cause                            |
|---------------|----------------------------------|
| 403 Forbidden | Missing permission `boards.post` |

--------------------------------------------------------------------------------

## {board_uri}
### GET
Gets a Board object.
#### Response
A Board Reference.
##### Headers
| Header                | Value                                                                          |
|-----------------------|--------------------------------------------------------------------------------|
| Pxls-Pixels-Available | Number of placements the client can create before being subject to a cooldown. |
| Pxls-Next-Available   | Timestamp of when `Pixels-Available` will increase.                            |
#### Errors
| Response Code | Cause                                  |
|---------------|----------------------------------------|
| 403 Forbidden | Missing permission `boards.get`        |

### PATCH
Updates the Board object.
#### Request
```typescript
Partial<{
	"name": string;
	"shape": number[][]
	"palette": Map<number, {
		"name": string;
		"value": number;
	}>;
}>
```
#### Response
A Board Reference.
#### Errors
| Response Code | Cause                             |
|---------------|-----------------------------------|
| 403 Forbidden | Missing permission `boards.patch` |

### DELETE
Deletes the Board object.
#### Response
*No Content*
#### Errors
| Response Code | Cause                              |
|---------------|------------------------------------|
| 403 Forbidden | Missing permission `boards.delete` |

--------------------------------------------------------------------------------

## /boards/default
Requests made to this endpoint should be redirected using HTTP status 307 to the board object of the default board.
If any sub-endpoints are called on this, the should likewise be redirected, keeping the path.

--------------------------------------------------------------------------------

## {board_uri}/socket?extensions[]={extensions_list}
Websocket connection point.
A connecting client may specify which extensions it wishes to be enabled on the websocket using the `extensions_list` query parameter.
Extension names are the same as those in `/info`.
To receive the events defined here, the list should contain the value `core`.
### Server packets
These packets are sent by the server to inform the client of something.
They should be sent as UTF-8 encoded JSON text.
#### BoardUpdate
The board has changed.
```typescript
{
	"type": "board-update";
	"info"?: Partial<{
		"name": string;
		"shape": number[][]
		"palette": Map<number, {
			"name": string;
			"value": number;
		}>;
	}>;
	"data"?: Partial<{
		"colors": Array<Change>;
		"timestamps": Array<Change>;
		"initial": Array<Change>;
		"mask": Array<Change>;
	}
}
```
Where a Change object is defined as:
```typescript
{
	"position": number;
	"values": number[];
}
```
Where `values` is a contiguous run of data and can be of any valid length.

In the vast majority of cases, this packet will look similar to the following, representing a placement event:
```json
{
	"type": "board-update",
	"data": {
		"colors": [
			{ "position": 0, "values": [1] }
		],
		"timestamps": [
			{ "position": 0, "values": [420] }
		]
	}
}
```
#### PixelsAvailable
How many pixels may be placed by the current user without encountering a cooldown.
```typescript
{
	"type": "pixels-available";
	"count": number;
	"next"?: Timestamp; 
}
```
### Errors
| Response Code            | Cause                               |
|--------------------------|-------------------------------------|
| 403 Forbidden            | Missing permission `socket.core`.   |
| 422 Unprocessable Entity | No extensions specified.            |
| 422 Unprocessable Entity | Requested extensions not supported. |

--------------------------------------------------------------------------------

## {board_uri}/data/initial
### GET
Represents the initial state of the board.
#### Response
Binary data. 
8-bit palette index for every pixel.
#### Errors
| Response Code | Cause                                 |
|---------------|---------------------------------------|
| 403 Forbidden | Missing permission `boards.data.get`. |

### PATCH
Update the initial board.
#### Request
Binary data.
Content-range can be specified.
Content-type may be multipart/byteranges which allows the client to patch smaller segments individually if the server supports it.
Server implementations must support content-range headers which specify a range aligning with shape boundaries, but need not support other range features.
#### Response
204 No Content
#### Errors
| Response Code | Cause                                   |
|---------------|-----------------------------------------|
| 403 Forbidden | Missing permission `boards.data.patch`. |
| 409 Conflict  | Patch is considered invalid.            |

--------------------------------------------------------------------------------

## {board_uri}/data/colors
### GET
Represents the current state of the board.
#### Response
Binary data. 
8-bit palette index for every pixel.
#### Errors
| Response Code | Cause                                 |
|---------------|---------------------------------------|
| 403 Forbidden | Missing permission `boards.data.get`. |

--------------------------------------------------------------------------------

## {board_uri}/data/mask
### GET
Represents where placements can be made within the board dimensions.
Values correspond to the following behaviors:
| Value | Placement Behavior                                        |
|:-----:|-----------------------------------------------------------|
|   0   | No placement allowed.                                     |
|   1   | Placement allowed.                                        |
|   2   | Placement allowed if an adjacent pixel has been modified. |

#### Response
Binary data. 
8-bit mask identifier for every pixel.
#### Errors
| Response Code | Cause                                 |
|---------------|---------------------------------------|
| 403 Forbidden | Missing permission `boards.data.get`. |

### PATCH
Update the board mask.
#### Request
Binary data.
Content-range can be specified.
Content-type may be multipart/byteranges which allows the client to patch smaller segments individually if the server supports it.
Server implementations must support content-range headers which specify a range aligning with shape boundaries, but need not support other range features.
#### Response
204 No Content
#### Errors
| Response Code | Cause                                   |
|---------------|-----------------------------------------|
| 403 Forbidden | Missing permission `boards.data.patch`. |
| 409 Conflict  | Patch is considered invalid.            |

--------------------------------------------------------------------------------

## {board_uri}/data/timestamps
### GET
Represents the last-modified time for all pixels on the board.
#### Response
Binary data. 
32-bit unsigned timestamp for every pixel. 
Bytes are little-endian ordered.
Timestamp is seconds since `created_at` as defined in `/board/info`.
#### Errors
| Response Code | Cause                                 |
|---------------|---------------------------------------|
| 403 Forbidden | Missing permission `boards.data.get`. |

--------------------------------------------------------------------------------

## {board_uri}/users
### GET
Gets the active user count.
#### Response
```typescript
{
	"active": number;
	"idle_timeout": number;
}
```
`idle_timeout` is in seconds.
#### Errors
| Response Code | Cause                              |
|---------------|------------------------------------|
| 403 Forbidden | Missing permission `boards.users`. |

--------------------------------------------------------------------------------

## {board_uri}/pixels
### GET
Lists all placements.
#### Response
A Paginated List of Placement objects, sorted by timestamp ascending.
#### Errors
| Response Code | Cause                                    |
|---------------|------------------------------------------|
| 403 Forbidden | Missing permission `boards.pixels.list`. |

--------------------------------------------------------------------------------

## {board_uri}/pixels/{position}
### GET
Gets the most recent placement for the specified board position.
#### Response
A Placement object.
#### Errors
| Response Code | Cause                                  |
|---------------|----------------------------------------|
| 404 Not Found | Position has no placements.            |
| 404 Not Found | Position outside of board dimensions.  |
| 403 Forbidden | Missing permission `board.pixels.get`. |

### POST
Creates a placement.
#### Request
```typescript
{
	"color": number;
}
```
#### Response
The created Placement object.
##### Headers
| Header                | Value                                                                          |
|-----------------------|--------------------------------------------------------------------------------|
| Pxls-Pixels-Available | Number of placements the client can create before being subject to a cooldown. |
| Pxls-Next-Available   | Timestamp of when `Pixels-Available` will increase.                            |
#### Errors
| Response Code            | Cause                                             |
|--------------------------|---------------------------------------------------|
| 404 Not Found            | Position outside of board dimensions.             |
| 403 Forbidden            | Missing permission `board.pixels.post`.           |
| 403 Forbidden            | Position is not placable according to board mask. |
| 409 Conflict             | Placement would have no effect.                   |
| 422 Unprocessable Entity | Color does not exist on the palette.              |
| 429 Too Many Requests    | No available pixels to place.                     |