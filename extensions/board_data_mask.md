Mask Board Data
===============
Implementing this extensions adds a new board data buffer, providing clients with information on where placements are allowed.

--------------------------------------------------------------------------------

## /info
### GET
#### Response
```typescript
{
	"extensions": ["board_mask"];
}
```

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

## {board_uri}/socket?extensions[]=board_mask
### Server packets
#### BoardUpdate
The board has changed.
```typescript
{
	"data"?: Partial<{
		"mask": Array<Change>;
	}
}
```
### Errors
| Response Code | Cause                                    |
|---------------|------------------------------------------|
| 403 Forbidden | Missing permission `socket.boards.mask`. |