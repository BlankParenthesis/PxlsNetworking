User Bans
=========
Implementing this extension provides a formal method to ban users, preventing them from making any modifying action on the board.
Since this concerns user objects directly, implementing this extension requires implementing the [users extension](./users.md).

This extension adds Ban objects which are described by following type:
```typescript
{
	"id": number | string;
	"issued": Timestamp;
	"expiry": Timestamp;
	"issuer"?: User;
	"reason": string;
}
```

--------------------------------------------------------------------------------

## /info
### GET
#### Response
```typescript
{
	"extensions": ["user_moderation"];
}
```

--------------------------------------------------------------------------------

## /board/pixels/{x}/{y}
### POST
#### Errors
| Response Code | Cause   |
|---------------|---------|
| 403 Forbidden | Banned. |

If the [board undo extension](./board_undo.md) is implemented, then undos are also unavailable due to bans:
### DELETE
#### Errors
| Response Code | Cause   |
|---------------|---------|
| 403 Forbidden | Banned. |

--------------------------------------------------------------------------------

If the [board moderation extension](./board_undo.md) is implemented, then mass-place events are also unavailable due to bans:
## /board/pixels
### PATCH
#### Errors
| Response Code | Cause   |
|---------------|---------|
| 403 Forbidden | Banned. |

--------------------------------------------------------------------------------

## /users/{user_id}/bans
### GET
Lists all bans issued to a user.
#### Response
A Paginated List of Ban objects.
##### Errors
| Response Code | Cause                                                              |
|---------------|--------------------------------------------------------------------|
| 403 Forbidden | Missing permission `users.get` or `users.current.get`.             |
| 404 Not Found | No such User exists.                                               |
| 403 Forbidden | Missing permission `users.bans.list` or `users.current.bans.list`. |

### POST
Bans a user.
#### Request
```typescript
{
	"expiry": Timestamp;
	"reason": string;
}
```
##### Response
The created Ban object.
##### Errors
| Response Code            | Cause                                                  |
|--------------------------|--------------------------------------------------------|
| 403 Forbidden            | Missing permission `users.get` or `users.current.get`. |
| 404 Not Found            | No such User exists.                                   |
| 403 Forbidden            | Missing permission `users.bans.post`.                  |
| 422 Unprocessable Entity | Invalid reason.                                        |

--------------------------------------------------------------------------------

## /users/{user_id}/bans/{ban_id}
### GET
Gets a ban issued to a user.
#### Response
A Ban object.
##### Errors
| Response Code | Cause                                                            |
|---------------|------------------------------------------------------------------|
| 403 Forbidden | Missing permission `users.get` or `users.current.get`.           |
| 404 Not Found | No such User exists.                                             |
| 403 Forbidden | Missing permission `users.bans.get` or `users.current.bans.get`. |
| 404 Not Found | No such Ban exists.                                              |

### PATCH
Updates a ban issued to a user.
#### Request
```typescript
{
	"expiry"?: Timestamp;
	"reason"?: string;
}
```
##### Response
The created Ban object.
##### Errors
| Response Code            | Cause                                                  |
|--------------------------|--------------------------------------------------------|
| 403 Forbidden            | Missing permission `users.get` or `users.current.get`. |
| 404 Not Found            | No such User exists.                                   |
| 403 Forbidden            | Missing permission `users.bans.patch`.                 |
| 404 Not Found            | No such Ban exists.                                    |
| 422 Unprocessable Entity | The ban reason is invalid.                             |

### DELETE
Deletes a ban issued to a user.
##### Errors
| Response Code | Cause                                                  |
|---------------|--------------------------------------------------------|
| 403 Forbidden | Missing permission `users.get` or `users.current.get`. |
| 404 Not Found | No such User exists.                                   |
| 403 Forbidden | Missing permission `users.bans.delete`.                |
| 404 Not Found | No such Ban exists.                                    |
