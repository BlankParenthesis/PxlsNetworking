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
```typescript
{
	"extensions": ["user_moderation"];
}
```

--------------------------------------------------------------------------------

## /board/pixels/{x}/{y}
### POST
#### Errors
| Response Code | Cause                                                   |
|---------------|---------------------------------------------------------|
| 403 Forbidden | One or more bans affect the client at the current time. |

If the [board undo extension](./board_undo.md) is implemented, then undos are also unavailable due to bans:
### DELETE
#### Errors
| Response Code | Cause                                                   |
|---------------|---------------------------------------------------------|
| 403 Forbidden | One or more bans affect the client at the current time. |

--------------------------------------------------------------------------------

If the [board moderation extension](./board_undo.md) is implemented, then mass-place events are also unavailable due to bans:
## /board/pixels
### PATCH
#### Errors
| Response Code | Cause                                                   |
|---------------|---------------------------------------------------------|
| 403 Forbidden | One or more bans affect the client at the current time. |

--------------------------------------------------------------------------------

## /users/{user_id}/bans
### GET
Information on every ban that has been issued to the specified user.
Returns a Paginated List of Ban objects.
##### Errors
| Response Code            | Cause                                             |
|--------------------------|---------------------------------------------------|
| 403 Forbidden            | The client lacks the permission `users.get`.      |
| 404 Not Found            | No user with the specified ID exists.             |
| 403 Forbidden            | The client lacks the permission `users.bans.list`.|

### POST
Bans the specified user.
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
| Response Code            | Cause                                             |
|--------------------------|---------------------------------------------------|
| 403 Forbidden            | The client lacks the permission `users.get`.      |
| 404 Not Found            | No user with the specified ID exists.             |
| 403 Forbidden            | The client lacks the permission `users.bans.post`.|
| 422 Unprocessable Entity | The ban reason is invalid.                        |

--------------------------------------------------------------------------------

## /users/{user_id}/bans/{ban_id}
### GET
Information on a specific ban that has been issued to the specified user.
Returns a Ban object.
##### Errors
| Response Code | Cause                                            |
|---------------|--------------------------------------------------|
| 403 Forbidden | The client lacks the permission `users.get`.     |
| 404 Not Found | No user with the specified ID exists.            |
| 403 Forbidden | The client lacks the permission `users.bans.get`.|
| 404 Not Found | No ban with the specified ID exists.             |

### PATCH
Updates a specific ban that has been issued to the specified user.
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
| Response Code            | Cause                                              |
|--------------------------|----------------------------------------------------|
| 403 Forbidden            | The client lacks the permission `users.get`.       |
| 404 Not Found            | No user with the specified ID exists.              |
| 403 Forbidden            | The client lacks the permission `users.bans.patch`.|
| 404 Not Found            | No ban with the specified ID exists.               |
| 422 Unprocessable Entity | The ban reason is invalid.                         |

### DELETE
Removes a specific ban that has been issued to the specified user.
##### Errors
| Response Code | Cause                                               |
|---------------|-----------------------------------------------------|
| 403 Forbidden | The client lacks the permission `users.get`.        |
| 404 Not Found | No user with the specified ID exists.               |
| 403 Forbidden | The client lacks the permission `users.bans.delete`.|
| 404 Not Found | No ban with the specified ID exists.                |
