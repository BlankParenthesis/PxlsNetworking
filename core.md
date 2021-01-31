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
	items: X[];
	next?: string;
	previous?: string;
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

--------------------------------------------------------------------------------

## /info
### GET
Information about the server implementation.
Extension definitions redefine the `extensions` field in this request.
Implementations should return the union of all lists defined by every implemented extensions as a set.
```typescript
{
	"name"?: string;
	"version"?: string;
	"source"?: string;
	"extensions": string[];
}
```

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
#### Errors
| Response Code            | Cause                                                                                                    |
|--------------------------|----------------------------------------------------------------------------------------------------------|
| 403 Forbidden            | The client lacks the required privileges to connect to the websocket with the specified extensions list. |
| 422 Unprocessable Entity | The server does not support all of the requested extensions.                                             |

--------------------------------------------------------------------------------

## /board/data/initial
### GET
Binary data. 
8-bit palette index for every pixel.
Represents the initial state of the board.
#### Errors
| Response Code | Cause                                                        |
|---------------|--------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to read board data. |

--------------------------------------------------------------------------------

## /board/data/colors
### GET
Binary data. 
8-bit palette index for every pixel.
Represents the current state of the board.
#### Errors
| Response Code | Cause                                                        |
|---------------|--------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to read board data. |

--------------------------------------------------------------------------------

## /board/data/mask
### GET
Binary data. 
8-bit mask identifier for every pixel.

 | Value | Placement Behavior                                        |
 |:-----:|-----------------------------------------------------------|
 |   0   | No placement allowed.                                     |
 |   1   | Placement allowed.                                        |
 |   2   | Placement allowed if an adjacent pixel has been modified. |

#### Errors
| Response Code | Cause                                                        |
|---------------|--------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to read board data. |

--------------------------------------------------------------------------------

## /board/data/modified
### GET
Binary data. 
32-bit timestamp for every pixel. 
Timestamp is seconds since `createdAt` as defined in `/board/info`.
Each timestamp represents the last-modified time for a pixel.

#### Errors
| Response Code | Cause                                                        |
|---------------|--------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to read board data. |

--------------------------------------------------------------------------------

## /board/info
### GET
Metadata for the current board.
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
}
```
#### Errors
| Response Code | Cause                                                        |
|---------------|--------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to read board data. |

--------------------------------------------------------------------------------

## /board/users
### GET
Information on the active and idle user counts.
```typescript
{
	"active": number;
	"idle": number;
	"idleTimeout": number;
}
```
#### Errors
| Response Code | Cause                                                             |
|---------------|-------------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to fetch the user count. |

--------------------------------------------------------------------------------

## /board/pixels
### GET
Information on all placements.
Returns a Paginated List of Placement objects.
#### Errors
| Response Code | Cause                                                        |
|---------------|--------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to list placements. |

--------------------------------------------------------------------------------

## /board/pixels/{x}/{y}
### GET
Information on the most recent placement for a given board position.
Returns a Placement object.
#### Errors
| Response Code            | Cause                                                        |
|--------------------------|--------------------------------------------------------------|
| 404 Not Found            | Not placement action has occurred at the specified position. |
| 404 Not Found            | The specified position is outside of the board dimensions.   |
| 403 Forbidden            | The client lacks the required privileges to read pixel data. |

### POST
Create a Placement for a given board coordinate.
#### Request
```typescript
{
	"color": number;
}
```
#### Response
```typescript
{
	"pixelsAvailable": number;
	"nextAvailable"?: Timestamp; 
}
```
#### Errors
| Response Code            | Cause                                                               |
|--------------------------|---------------------------------------------------------------------|
| 404 Not Found            | The specified position is outside of the board dimensions.          |
| 403 Forbidden            | The client is banned from placing.                                  |
| 403 Forbidden            | The client does not have the required privileges to post pixels.    |
| 403 Forbidden            | The specified position is not placable according to the board mask. |
| 409 Conflict             | The change would have no effect.                                    |
| 422 Unprocessable Entity | The supplied color does not exist on the palette.                   |
| 429 Too Many Requests    | The client user has no available pixels to place.                   |