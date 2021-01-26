Users
=====
Implementing this extension attaches a unique persistent profile to every authenticated user.
Consequently, implementing this extension requires implementing the [authentication extension](./authentication.md).

User objects are represented with the following type:
```typescript
{
	"name": string;
	"createdAt": Timestamp;
}
```

If the [roles extension](./roles.md) is implemented, the following permissions are added due to this extension:

| Permission         | Purpose                                                                                            |
|--------------------|----------------------------------------------------------------------------------------------------|
| `users.list`       | Allows GET requests to `/users`.                                                                   |
| `users.get`        | Allows GET requests to `/users/{user_id}`.                                                         |
| `users.patch`      | Allows PATCH requests to `/users/{user_id}`.                                                       |
| `users.patch.name` | Allows PATCH requests for `name` to `/users/{user_id}` where `{user_id}` is the current user's ID. |

--------------------------------------------------------------------------------

## /info
### GET
```typescript
{
	"extensions": ["users"];
}
```

--------------------------------------------------------------------------------

## /board/ws
### Server packets
#### UserUpdate
The client user has changed.
```typescript
{
	"type": "user-updated";
	"user": Partial<User>;
}
```

--------------------------------------------------------------------------------

## /board/pixels/{x}/{y}
### GET
```typescript
{
	"user"?: User;
}
```

--------------------------------------------------------------------------------

## /users
### GET
A list of all users.
#### Response
An array of User References.
#### Errors
| Response Code | Cause                                                           |
|---------------|-----------------------------------------------------------------|
| 403 Forbidden | The client does not have the required privileges to list users. |

--------------------------------------------------------------------------------

## {user_uri}
### GET
#### Response
The User object.
#### Errors
| Response Code | Cause                                                                     |
|---------------|---------------------------------------------------------------------------|
| 403 Forbidden | The client does not have the required privileges to view the user's data. |

### PATCH
Updates the User object.
#### Request
A partial User object.
#### Response
The updated User object.
#### Errors
| Response Code | Cause                                                                     |
|---------------|---------------------------------------------------------------------------|
| 403 Forbidden | The client does not have the required privileges to edit the user's data. |


