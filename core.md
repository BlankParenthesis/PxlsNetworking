Core Specification
==================
All timestamps are Unix time unless specified otherwise. 
They should be treated as the number type in practice but are represented clearly as "Timestamp" here.

A number of core protocol extensions exist. 
Within these extensions, endpoints declared either here or in other extensions may be re-declared.
These **duplicate response type definitions should be treated as the union of implemented definitions**.
This allows extensions to add new fields to responses in a flexible way.

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

## /board/ws
Websocket connection point.
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
| Response Code | Cause                                                                 |
|---------------|-----------------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to connect to the websocket. |

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
Timestamp is seconds since board creation.
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

## /board/pixels/{x}/{y}
### GET
Information for a given board position.
```typescript
{
	"color": number;
	"modified": Timestamp;
}
```
#### Errors
| Response Code            | Cause                                                        |
|--------------------------|--------------------------------------------------------------|
| 404 Not Found            | The specified position is outside of the board dimensions.   |
| 403 Forbidden            | The client lacks the required privileges to read pixel data. |

### POST
Replace the pixel at a given board coordinate.
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