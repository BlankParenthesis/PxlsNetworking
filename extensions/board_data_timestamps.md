Timestamps Board Data
=====================
Implementing this extensions adds a new board data buffer, providing clients with information on the timestamp of the most recent placement for every pixel.

--------------------------------------------------------------------------------

## Endpoints

### /info
#### GET
##### Response
```typescript
{
	"extensions": ["board_timestamps"];
}
```

--------------------------------------------------------------------------------

### {board_uri}/data/timestamps
#### GET
Represents the last-modified time for all pixels on the board.
##### Response
Binary data. 
32-bit unsigned timestamp for every pixel. 
Bytes are little-endian ordered.
Timestamp is seconds since `created_at` as defined in `{board_uri}/info`.
##### Errors
| Response Code | Cause                                 |
|---------------|---------------------------------------|
| 403 Forbidden | Missing permission `boards.data.get`. |

--------------------------------------------------------------------------------

### {board_uri}/socket?extensions[]=board_timestamps
#### Server packets
##### BoardUpdate
The board has changed.
```typescript
{
	"data"?: Partial<{
		"timestamps": Array<Change>;
	}
}
```
#### Errors
| Response Code | Cause                                          |
|---------------|------------------------------------------------|
| 403 Forbidden | Missing permission `socket.boards.timestamps`. |