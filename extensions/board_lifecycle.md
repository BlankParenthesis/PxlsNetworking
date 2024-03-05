Board Lifecycle Management
==========================
Implementing this extensions allows clients to update the board metadata.
*NOTE: Server implementations should disconnect socket connections on metadata changes if that connection does not have this extension enabled.*

--------------------------------------------------------------------------------

## Endpoints

### /info
#### GET
##### Response
```typescript
{
	"extensions": ["board_lifecycle"];
}
```

--------------------------------------------------------------------------------

### /events?subscribe[]={events_list}
#### Server packets
##### BoardCreated
Sent when a board is created.
Set `subscribe[]=boards` to add this field to the event.
```typescript
{
	"type": "board-created";
	"board": Reference<Board>;
}
```
##### BoardRemoved
Sent when a board is deleted.
Set `subscribe[]=boards` to add this field to the event.
```typescript
{
	"type": "board-removed";
	"board": Url;
}
```
#### Errors
| Response Code | Cause                               |
|---------------|-------------------------------------|
| 403 Forbidden | Missing permission `events.boards`. |

--------------------------------------------------------------------------------

### {board_uri}
#### POST
Creates a new Board object.
##### Request
```typescript
{
	"name": string;
	"shape": number[][]
	"palette": Map<number, {
		"name": string;
		"value": number;
	}>;
	"max_pixels_available": number;
}
```
##### Response
A Board Reference.
###### Headers
| Header   | Value                              |
|----------|------------------------------------|
| Location | The location of the created Board. |
##### Errors
| Response Code | Cause                             |
|---------------|-----------------------------------|
| 403 Forbidden | Missing permission `boards.post`. |

#### PATCH
Updates the Board object.
##### Request
```typescript
Partial<{
	"name": string;
	"shape": number[][]
	"palette": Map<number, {
		"name": string;
		"value": number;
	}>;
	"max_pixels_available": number;
}>
```
##### Response
A Board Reference.
##### Errors
| Response Code | Cause                              |
|---------------|------------------------------------|
| 403 Forbidden | Missing permission `boards.patch`. |

#### DELETE
Deletes the Board object.
##### Response
*No Content*
##### Errors
| Response Code | Cause                               |
|---------------|-------------------------------------|
| 403 Forbidden | Missing permission `boards.delete`. |

--------------------------------------------------------------------------------

### {board_uri}/events?subscribe[]={events_list}
#### Server packets
##### BoardUpdate
Set `subscribe[]=info` to add this field to the event.
```typescript
{
	"info"?: Partial<{
		"name": string;
		"shape": number[][];
		"palette": Map<number, {
			"name": string;
			"value": number;
		}>;
	}>;
}
```
#### Errors
| Response Code | Cause                                    |
|---------------|------------------------------------------|
| 403 Forbidden | Missing permission `boards.events.info`. |