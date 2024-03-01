Initial Board Data
==================
Implementing this extensions adds a new board data buffer, providing clients with information on the initial state of the board before any placements occurred.

--------------------------------------------------------------------------------

## /info
### GET
#### Response
```typescript
{
	"extensions": ["board_initial"];
}
```

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

## {board_uri}/socket?extensions[]=board_initial
### Server packets
#### BoardUpdate
The board has changed.
```typescript
{
	"data"?: Partial<{
		"initial": Array<Change>;
	}
}
```
### Errors
| Response Code | Cause                                       |
|---------------|---------------------------------------------|
| 403 Forbidden | Missing permission `socket.boards.initial`. |