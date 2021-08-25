User Bans
=========
Implementing this extension provides a formal method to ban users, preventing them from making any modifying action on the board.
Since this concerns user objects directly, implementing this extension requires implementing the [users extension](./users.md).

This extension adds Ban objects which are described by following type:
```typescript
{
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

## /board/pixels/{x}/{y?}/{z?}/…
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

## {user_uri}/bans
### GET
Lists all bans issued to a user.
#### Response
A Paginated List of Ban References.
##### Errors
| Response Code | Cause                                                              |
|---------------|--------------------------------------------------------------------|
| 403 Forbidden | Missing permission `users.get` or `users.current.get`.             |
| 403 Forbidden | Missing permission `users.bans.list` or `users.current.bans.list`. |

### POST
Bans the user.
#### Request
```typescript
{
	"expiry": Timestamp;
	"reason": string;
}
```
##### Response
A reference to the created Ban object.
##### Errors
| Response Code            | Cause                                                  |
|--------------------------|--------------------------------------------------------|
| 403 Forbidden            | Missing permission `users.get` or `users.current.get`. |
| 403 Forbidden            | Missing permission `users.bans.post`.                  |
| 422 Unprocessable Entity | Invalid reason.                                        |

--------------------------------------------------------------------------------

## {ban_uri}
### GET
Gets a ban issued to a user.
#### Response
The Ban object.
##### Errors
| Response Code | Cause                                                            |
|---------------|------------------------------------------------------------------|
| 403 Forbidden | Missing permission `users.get` or `users.current.get`.           |
| 403 Forbidden | Missing permission `users.bans.get` or `users.current.bans.get`. |

### PATCH
Updates the ban.
#### Request
```typescript
{
	"expiry"?: Timestamp;
	"reason"?: string;
}
```
##### Response
The updated Ban object.
##### Errors
| Response Code            | Cause                                                  |
|--------------------------|--------------------------------------------------------|
| 403 Forbidden            | Missing permission `users.get` or `users.current.get`. |
| 403 Forbidden            | Missing permission `users.bans.patch`.                 |
| 422 Unprocessable Entity | The ban reason is invalid.                             |

### DELETE
Removes the ban.
##### Errors
| Response Code | Cause                                                  |
|---------------|--------------------------------------------------------|
| 403 Forbidden | Missing permission `users.get` or `users.current.get`. |
| 403 Forbidden | Missing permission `users.bans.delete`.                |
