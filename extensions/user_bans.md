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

## {user_uri}/bans
### GET
A list of all bans for the user.
#### Response
An array of Ban References.
##### Errors
| Response Code            | Cause                                                          |
|--------------------------|----------------------------------------------------------------|
| 403 Forbidden            | The client does not have the required privileges to list bans. |

### POST
Ban the user.
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
| Response Code            | Cause                                                             |
|--------------------------|-------------------------------------------------------------------|
| 403 Forbidden            | The client does not have the required privileges to ban the user. |
| 422 Unprocessable Entity | The ban reason is invalid.                                        |

--------------------------------------------------------------------------------

## {ban_uri}
### GET
#### Response
The Ban object.
##### Errors
| Response Code | Cause                                                          |
|---------------|----------------------------------------------------------------|
| 403 Forbidden | The client does not have the required privileges view the ban. |

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
| Response Code            | Cause                                                             |
|--------------------------|-------------------------------------------------------------------|
| 403 Forbidden            | The client does not have the required privileges to ban the user. |
| 422 Unprocessable Entity | The ban reason is invalid.                                        |

### DELETE
Removes the ban.
##### Errors
| Response Code | Cause                                                               |
|---------------|---------------------------------------------------------------------|
| 403 Forbidden | The client does not have the required privileges to unban the user. |
