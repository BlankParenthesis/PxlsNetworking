Board Moderation
================
Implementing this extension provides users with the tools necessary to moderate the content of the board directly.

--------------------------------------------------------------------------------

## /info
### GET
#### Response
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
| Response Code | Cause                                                |
|---------------|------------------------------------------------------|
| 403 Forbidden | Missing permission `board.pixels.override.cooldown`. |
| 403 Forbidden | Missing permission `board.pixels.override.color`.    |
| 403 Forbidden | Missing permission `board.pixels.override.mask`.     |

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
}
```
##### Headers
| Header           | Value                                                                          |
|------------------|--------------------------------------------------------------------------------|
| Pixels-Available | Number of placements the client can create before being subject to a cooldown. |
| Next-Available   | Timestamp of when `Pixels-Available` will increase.                            |

#### Errors
| Response Code            | Cause                                                |
|--------------------------|------------------------------------------------------|
| 403 Forbidden            | Missing permission `board.pixels.patch`.             |
| 404 Not Found            | Position outside of board dimensions.                |
| 403 Forbidden            | Missing permission `board.pixels.override.cooldown`. |
| 403 Forbidden            | Missing permission `board.pixels.override.color`.    |
| 403 Forbidden            | Missing permission `board.pixels.override.mask`.     |
| 403 Forbidden            | Position is not placable according to board mask.    |
| 409 Conflict             | Placement would have no effect.                      |
| 422 Unprocessable Entity | Color does not exist on the palette.                 |
| 429 Too Many Requests    | No available pixels to place.                        |