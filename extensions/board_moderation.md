Board Moderation
================
Implementing this extension provides users with the tools necessary to moderate the content of the board directly.

If the [roles extension](./roles.md) is implemented, the following permissions are added due to this extension:

| Permission                       | Purpose                                                     |
|----------------------------------|-------------------------------------------------------------|
| `board.pixels.override.cooldown` | Allows sending requests with the cooldown override enabled. |
| `board.pixels.override.color`    | Allows sending requests with the color override enabled.    |
| `board.pixels.override.mask`     | Allows sending requests with the mask override enabled.     |
| `board.pixels.patch`             | Allows sending PATCH requests to `/board/pixels`.           |

--------------------------------------------------------------------------------

## /info
### GET
```typescript
{
	"extensions": ["board_moderation"];
}
```

--------------------------------------------------------------------------------

## /board/pixels/{x}/{y}
### POST
#### Request
```typescript
{
	"overrides"?:  {
		"cooldown": boolean;
		"color": boolean;
		"mask": boolean;
	}
}
```
#### Errors
| Response Code | Cause                                                                            |
|---------------|----------------------------------------------------------------------------------|
| 403 Forbidden | The client does not have the required privileges to use all specified overrides. |

--------------------------------------------------------------------------------

## /board/pixels
### PATCH
Replace a subarea of the board.
This is most useful for when moderators need to censor or otherwise change a large area and other tools - such as user rollbacks - are insufficient.
Since this obeys override rules, permission for this endpoint can be granted to regular users.
If a user has not specified a cooldown override and has some but not all of the available pixels to make the action, the action should be rejected outright.
#### Request
```typescript
{
	"x": number;
	"y": number;
	"width": number;
	"height": number;
	"colors": string;
	"overrides"?:  {
		"cooldown": boolean;
		"color": boolean;
		"mask": boolean;
	}
}
```
#### Response
```typescript
{
	"pixelsChanged": number;
	"pixelsAvailable": number;
	"nextAvailable"?: Timestamp; 
}
```
#### Errors
| Response Code            | Cause                                                                            |
|--------------------------|----------------------------------------------------------------------------------|
| 404 Not Found            | The specified position is outside of the board dimensions.                       |
| 403 Forbidden            | The client is banned from placing.                                               |
| 403 Forbidden            | The client does not have the required privileges to mass post pixels.            |
| 403 Forbidden            | The specified position is not placable according to the board mask.              |
| 403 Forbidden            | The client does not have the required privileges to use all specified overrides. |
| 409 Conflict             | The change would have no effect.                                                 |
| 422 Unprocessable Entity | The supplied color does not exist on the palette.                                |
| 429 Too Many Requests    | The client user has less pixels available than needed to fulfil the request.     |
