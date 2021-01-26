Chat Mutes
==========
Implementing this provides a formal method to mute users, preventing them from taking any modifying action in chat or posting new messages.
Mutes are similar to bans but affect chat rather than board actions (see the [bans extension](./user_bans.md)).
Since this concerns chat directly, implementing this extension requires implementing the [chat extension](./chat.md).
It also requires implementing the [users extension](./users.md) to identify users.

This extension adds Mute objects which are described by following type:
```typescript
{
	"id": number | string;
	"issued": Timestamp;
	"expiry": Timestamp;
	"issuer"?: User;
	"reason": string;
}
```

This object appears identical to the Ban object described in the [bans extension](./user_bans.md).

If the [roles extension](./roles.md) is implemented, the following permissions are added due to this extension:

| Permission           | Purpose                                                       |
|----------------------|---------------------------------------------------------------|
| `users.mutes.list`   | Allows GET requests to `/users/{user_id}/mutes`.              |
| `users.mutes.get`    | Allows GET requests to `/users/{user_id}/mutes/{mute_id}`.    |
| `users.mutes.post`   | Allows POST requests to `/users/{user_id}/mutes`.             |
| `users.mutes.patch`  | Allows PATCH requests to `/users/{user_id}/mutes/{mute_id}`.  |
| `users.mutes.delete` | Allows DELETE requests to `/users/{user_id}/mutes/{mute_id}`. |

--------------------------------------------------------------------------------

## /info
### GET
```typescript
{
	"extensions": ["chat_mutes"];
}
```

--------------------------------------------------------------------------------

## /chat/rooms/{room_id}/messages
### POST
#### Errors
| Response Code | Cause                                                    |
|---------------|----------------------------------------------------------|
| 403 Forbidden | One or more mutes affect the client at the current time. |

--------------------------------------------------------------------------------

## /users/{user_id}/mutes
### GET
Information on every mute that has been issued to the specified user.
Returns an array of Mute objects.
##### Errors
| Response Code            | Cause                                                           |
|--------------------------|-----------------------------------------------------------------|
| 404 Not Found            | No user with the specified ID exists.                           |
| 403 Forbidden            | The client does not have the required privileges to list mutes. |

### POST
Mutes the specified user.
#### Request
```typescript
{
	"expiry": Timestamp;
	"reason": string;
}
```
##### Response
The created Mute object.
##### Errors
| Response Code            | Cause                                                              |
|--------------------------|--------------------------------------------------------------------|
| 404 Not Found            | No user with the specified ID exists.                              |
| 403 Forbidden            | The client does not have the required privileges to mute the user. |
| 422 Unprocessable Entity | The mute reason is invalid.                                        |

--------------------------------------------------------------------------------

## /users/{user_id}/mutes/{mute_id}
### GET
Information on a specific mute that has been issued to the specified user.
Returns a Mute object.
##### Errors
| Response Code | Cause                                                           |
|---------------|-----------------------------------------------------------------|
| 404 Not Found | No user with the specified ID exists.                           |
| 404 Not Found | No mute with the specified ID exists.                           |
| 403 Forbidden | The client does not have the required privileges view the mute. |

### PATCH
Updates a specific mute that has been issued to the specified user.
#### Request
```typescript
{
	"expiry"?: Timestamp;
	"reason"?: string;
}
```
##### Response
The created Mute object.
##### Errors
| Response Code            | Cause                                                              |
|--------------------------|--------------------------------------------------------------------|
| 404 Not Found            | No user with the specified ID exists.                              |
| 404 Not Found            | No mute with the specified ID exists.                              |
| 403 Forbidden            | The client does not have the required privileges to mute the user. |
| 422 Unprocessable Entity | The mute reason is invalid.                                        |

### DELETE
Removes a specific mute that has been issued to the specified user.
##### Errors
| Response Code | Cause                                                                |
|---------------|----------------------------------------------------------------------|
| 404 Not Found | No user with the specified ID exists.                                |
| 404 Not Found | No mute with the specified ID exists.                                |
| 403 Forbidden | The client does not have the required privileges to unmute the user. |