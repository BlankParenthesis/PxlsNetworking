Chat Mutes
==========
Implementing this provides a formal method to mute users, preventing them from taking any modifying action in chat or posting new messages.
Mutes are similar to bans but affect chat rather than board actions (see the [bans extension](./user_bans.md)).
Since this concerns chat directly, implementing this extension requires implementing the [chat extension](./chat.md).
It also requires implementing the [users extension](./users.md) to identify users.

This extension adds Mute objects which are described by following type:
```typescript
{
	"issued": Timestamp;
	"expiry": Timestamp;
	"issuer"?: User;
	"reason": string;
}
```

This object appears identical to the Ban object described in the [bans extension](./user_bans.md).

--------------------------------------------------------------------------------

## /info
### GET
#### Response
```typescript
{
	"extensions": ["chat_mutes"];
}
```

--------------------------------------------------------------------------------

## {chatroom_uri}/messages
### POST
#### Errors
| Response Code | Cause  |
|---------------|--------|
| 403 Forbidden | Muted. |

--------------------------------------------------------------------------------

## {user_uri}/mutes
### GET
Lists all mutes issued to a user.
#### Response
A Paginated List of Mute references.
##### Errors
| Response Code | Cause                                                                |
|---------------|----------------------------------------------------------------------|
| 403 Forbidden | Missing permission `users.mutes.list` or `users.current.mutes.list`. |

### POST
Mutes a user.
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
| Response Code            | Cause                                   |
|--------------------------|-----------------------------------------|
| 403 Forbidden            | Missing permission `users.mutes.post`.  |
| 422 Unprocessable Entity | Invalid reason.                         |

--------------------------------------------------------------------------------

## {mute_uri}
### GET
Gets a mute issued to a user.
#### Response
The Mute object.
##### Errors
| Response Code | Cause                                                              |
|---------------|--------------------------------------------------------------------|
| 403 Forbidden | Missing permission `users.mutes.get` or `users.current.mutes.get`. |

### PATCH
Updates a mute issued to a user.
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
| Response Code            | Cause                                   |
|--------------------------|-----------------------------------------|
| 403 Forbidden            | Missing permission `users.mutes.patch`. |
| 422 Unprocessable Entity | Invalid reason.                         |

### DELETE
Removes a the Mute.
##### Errors
| Response Code | Cause                                    |
|---------------|------------------------------------------|
| 403 Forbidden | Missing permission `users.mutes.delete`. |
