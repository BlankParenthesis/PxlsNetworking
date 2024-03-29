Board Undo
==========
Implementing this extension allows users to retract placements - potentially for a refund of available pixels.
Servers may choose to limit when an undo action can take place.
This extension accommodates specifying a time-deadline for undo actions but does not consider required order of retractions or counted limits.
Clients are expected to keep track of which actions can be retracted.

--------------------------------------------------------------------------------

## Endpoints

### /info
#### GET
##### Response
```typescript
{
	"extensions": ["undo"];
}
```

--------------------------------------------------------------------------------

### {board_uri}/pixels/{position}
#### POST
##### Response
###### Headers
| Header             | Value                                                                      |
|--------------------|----------------------------------------------------------------------------|
| Pxls-Undo-Deadline | Timestamp before which DELETE actions can be performed for this placement. |

#### DELETE
Undoes the last place action at the given coordinate.
##### Response
###### Headers
| Header                | Value                                                                          |
|-----------------------|--------------------------------------------------------------------------------|
| Pxls-Pixels-Available | Number of placements the client can create before being subject to a cooldown. |
| Pxls-Next-Available   | Timestamp of when `Pixels-Available` will increase.                            |
##### Errors
| Response Code | Cause                                   |
|---------------|-----------------------------------------|
| 403 Forbidden | Missing permission `board.pixels.undo`. |
| 404 Not Found | Position outside of board dimensions.   |
| 403 Forbidden | Pixel not placed by the client user.    | 
| 409 Conflict  | `undo_deadline` for this pixel expired.  |

--------------------------------------------------------------------------------

If the [board moderation extension](./board_moderation.md) is implemented, then the mass-place endpoint defined there also gets an undo timestamp sent back:
### {board_uri}/pixels
#### PATCH
##### Response
###### Headers
| Header             | Value                                                                      |
|--------------------|----------------------------------------------------------------------------|
| Pxls-Undo-Deadline | Timestamp of when DELETE actions can no longer be sent for this placement. |