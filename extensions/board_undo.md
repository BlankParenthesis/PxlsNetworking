Board Undo
==========
Implementing this extension allows users to retract placements - potentially for a refund of available pixels.
Servers may choose to limit when an undo action can take place.
This extension accommodates specifying a time-deadline for undo actions but does not consider required order of retractions or counted limits.
Clients are expected to keep track of which actions can be retracted.

--------------------------------------------------------------------------------

## /info
### GET
#### Response
```typescript
{
	"extensions": ["undo"];
}
```

--------------------------------------------------------------------------------

## /board/pixels/{x}/{y}
### POST
#### Response
```typescript
{
	"undoDeadline": Timestamp;
}
```

### DELETE
Undoes the last place action at the given coordinate.
#### Response
```typescript
{
	"pixelsAvailable": number;
	"nextAvailable"?: Timestamp; 
}
```
#### Errors
| Response Code | Cause                                   |
|---------------|-----------------------------------------|
| 403 Forbidden | Missing permission `board.pixels.undo`. |
| 404 Not Found | Position outside of board dimensions.   |
| 403 Forbidden | Pixel not placed by the client user.    | 
| 409 Conflict  | `undoDeadline` for this pixel expired.  |

--------------------------------------------------------------------------------

If the [board moderation extension](./board_moderation.md) is implemented, then the mass-place endpoint defined there also gets an undo timestamp sent back:
## /board/pixels
### PATCH
#### Response
```typescript
{
	"undoDeadline": Timestamp;
}
```