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

If the [roles extension](./roles.md) is implemented, the following permissions are added due to this extension:

| Permission          | Purpose                                                     |
|---------------------|-------------------------------------------------------------|
| `users.bans.list`   | Allows GET requests to `/users/{user_id}/bans`.             |
| `users.bans.get`    | Allows GET requests to `/users/{user_id}/bans/{ban_id}`.    |
| `users.bans.post`   | Allows POST requests to `/users/{user_id}/bans`.            |
| `users.bans.patch`  | Allows PATCH requests to `/users/{user_id}/bans/{ban_id}`.  |
| `users.bans.delete` | Allows DELETE requests to `/users/{user_id}/bans/{ban_id}`. |


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

--------------------------------------------------------------------------------

## /users/{user_id}/bans
### GET
Information on every ban that has been issued to the specified user.
Returns an array of Ban objects.
##### Errors
| Response Code            | Cause                                                          |
|--------------------------|----------------------------------------------------------------|
| 404 Not Found            | No user with the specified ID exists.                          |
| 403 Forbidden            | The client does not have the required privileges to list bans. |

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
| Response Code            | Cause                                                             |
|--------------------------|-------------------------------------------------------------------|
| 404 Not Found            | No user with the specified ID exists.                             |
| 403 Forbidden            | The client does not have the required privileges to ban the user. |
| 422 Unprocessable Entity | The ban reason is invalid.                                        |

--------------------------------------------------------------------------------

## /users/{user_id}/bans/{ban_id}
### GET
Information on a specific ban that has been issued to the specified user.
Returns a Ban object.
##### Errors
| Response Code | Cause                                                          |
|---------------|----------------------------------------------------------------|
| 404 Not Found | No user with the specified ID exists.                          |
| 404 Not Found | No ban with the specified ID exists.                           |
| 403 Forbidden | The client does not have the required privileges view the ban. |

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
| Response Code            | Cause                                                             |
|--------------------------|-------------------------------------------------------------------|
| 404 Not Found            | No user with the specified ID exists.                             |
| 404 Not Found            | No ban with the specified ID exists.                              |
| 403 Forbidden            | The client does not have the required privileges to ban the user. |
| 422 Unprocessable Entity | The ban reason is invalid.                                        |

### DELETE
Removes a specific ban that has been issued to the specified user.
##### Errors
| Response Code | Cause                                                               |
|---------------|---------------------------------------------------------------------|
| 404 Not Found | No user with the specified ID exists.                               |
| 404 Not Found | No ban with the specified ID exists.                                |
| 403 Forbidden | The client does not have the required privileges to unban the user. |
