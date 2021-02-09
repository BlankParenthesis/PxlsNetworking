Core Specification
==================
All timestamps are Unix time unless specified otherwise. 
They should be treated as the number type in practice but are represented clearly as "Timestamp" here.

A number of core protocol extensions exist. 
Within these extensions, endpoints declared either here or in other extensions may be re-declared.
These **duplicate response type definitions should be treated as the union of implemented definitions**.
This allows extensions to add new fields to responses in a flexible way.

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


At the core of pxls is the ability to modify the board with Placements.
A Placement object represents a change of board state at a particular time and is defined by the following type:
```typescript
{
	"x": number;
	"y": number;
	"color": number;
	"modified": Timestamp;
}
```
Placements are often not functional distinguishable from pixels on the board and the two concepts are mostly treated interchangeably.


Nearly all requests by the client can be rejected by the server implementation.
It is useful for the client to know before making such requests if it will be allowed to do so.
For this reason, each method of each endpoint is associated with a string denoting the permission to use that endpoint.
A list of these strings is sent to the client in the `/info` endpoint.
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
	"permissions": string[];
}
```
Extension definitions redefine the `extensions` field in this request.
Implementations should return the union of all lists defined by every implemented extensions as a set.

The `permissions` field  contains a list of permissions strings.
It represents the actions the client can take without encountering a permissions error.

--------------------------------------------------------------------------------

## /ws?extensions[]={extensions_list}
Websocket connection point.
A connecting client may specify which extensions it wishes to be enabled on the websocket using the `extensions_list` query parameter.
Extension names are the same as those in `/info`.
To receive the events defined here, the list should contain the value `core`.
### Server packets
These packets are sent by the server to inform the client of something.
#### BoardUpdate
The board has changed.
```typescript
{
	"type": "board-update";
	"pixels": Array<{
		"x": number;
		"y": number;
		"color": number;
	}>;
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
#### Permissions
The client's permissions have changed.
```typescript
{
	"permissions": string[];
}
```
### Errors
| Response Code            | Cause                               |
|--------------------------|-------------------------------------|
| 403 Forbidden            | Missing permission `socket.core`.   |
| 422 Unprocessable Entity | No extensions specified.            |
| 422 Unprocessable Entity | Requested extensions not supported. |

--------------------------------------------------------------------------------

## /board/data/initial
### GET
Represents the initial state of the board.
#### Response
Binary data. 
8-bit palette index for every pixel.
Board data is ordered left-to-right, top-to-bottom.
#### Errors
| Response Code | Cause                            |
|---------------|----------------------------------|
| 403 Forbidden | Missing permission `board.data`. |

--------------------------------------------------------------------------------

## /board/data/colors
### GET
Represents the current state of the board.
#### Response
Binary data. 
8-bit palette index for every pixel.
Board data is ordered left-to-right, top-to-bottom.
#### Errors
| Response Code | Cause                            |
|---------------|----------------------------------|
| 403 Forbidden | Missing permission `board.data`. |

--------------------------------------------------------------------------------

## /board/data/mask
### GET
Represents where placements can be made withing the board dimensions.
Values correspond to the following behaviors:
| Value | Placement Behavior                                        |
|:-----:|-----------------------------------------------------------|
|   0   | No placement allowed.                                     |
|   1   | Placement allowed.                                        |
|   2   | Placement allowed if an adjacent pixel has been modified. |

#### Response
Binary data. 
8-bit mask identifier for every pixel.
Board data is ordered left-to-right, top-to-bottom.
#### Errors
| Response Code | Cause                            |
|---------------|----------------------------------|
| 403 Forbidden | Missing permission `board.data`. |

--------------------------------------------------------------------------------

## /board/data/modified
### GET
Represents the last-modified time for all pixels on the board.
#### Response
Binary data. 
32-bit timestamp for every pixel. 
Bytes are little-endian ordered.
Board data is ordered left-to-right, top-to-bottom.
Timestamp is seconds since `createdAt` as defined in `/board/info`.
#### Errors
| Response Code | Cause                            |
|---------------|----------------------------------|
| 403 Forbidden | Missing permission `board.data`. |

--------------------------------------------------------------------------------

## /board/info
### GET
Metadata for the current board.
#### Response
```typescript
{
	"name": string;
	"createdAt": Timestamp;
	"width": number;
	"height": number;
	"palette": Array<{
		"name": string;
		"value": string;
	}>;
	"maxPixelsAvailable": number;
}
```
##### Headers
| Header                | Value                                                                          |
|-----------------------|--------------------------------------------------------------------------------|
| Pxls-Pixels-Available | Number of placements the client can create before being subject to a cooldown. |
| Pxls-Next-Available   | Timestamp of when `Pixels-Available` will increase.                            |
#### Errors
| Response Code | Cause                            |
|---------------|----------------------------------|
| 403 Forbidden | Missing permission `board.data`. |

--------------------------------------------------------------------------------

## /board/users
### GET
Gets the active and idle user counts.
#### Response
```typescript
{
	"active": number;
	"idle": number;
	"idleTimeout": number;
}
```
#### Errors
| Response Code | Cause                             |
|---------------|-----------------------------------|
| 403 Forbidden | Missing permission `board.users`. |

--------------------------------------------------------------------------------

## /board/pixels
### GET
Lists all placements.
#### Response
A Paginated List of Placement objects.
#### Errors
| Response Code | Cause                                   |
|---------------|-----------------------------------------|
| 403 Forbidden | Missing permission `board.pixels.list`. |

--------------------------------------------------------------------------------

## /board/pixels/{x}/{y}
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