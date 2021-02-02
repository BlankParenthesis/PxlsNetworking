Users
=====
Implementing this extension attaches a unique persistent profile to every authenticated user.
Consequently, implementing this extension requires implementing the [authentication extension](./authentication.md).

User objects are represented with the following type:
```typescript
{
	"id": number | string;
	"name": string;
	"createdAt": Timestamp;
}
```

Placement objects gain an additional field due to this extension:
```typescript
{
	"user"?: User;
}
```

If the [roles extension](./roles.md) is implemented, the following permissions are added due to this extension:

| Permission          | Purpose                                                                                  |
|---------------------|------------------------------------------------------------------------------------------|
| `socket.users`      | Allows connecting to the websocket at `/ws` with `users` in the extensions list.         |
| `users.list`        | Allows GET requests to `/users`.                                                         |
| `users.get`         | Allows GET requests to `/users/{user_id}`.                                               |
| `users.patch`       | Allows PATCH requests to `/users/{user_id}`.                                             |
| `users.delete`      | Allows DELETE requests `/users/{user_id}`.                                               |
| `users.self.patch`  | Allows PATCH requests to `/users/{user_id}` where `{user_id}` is the current user's ID.  |
| `users.self.delete` | Allows DELETE requests to `/users/{user_id}` where `{user_id}` is the current user's ID. |

--------------------------------------------------------------------------------

## /info
### GET
```typescript
{
	"extensions": ["users"];
}
```

--------------------------------------------------------------------------------

## /ws?extensions[]=users
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

## /users
### GET
Returns a Paginated List of User objects.
#### Errors
| Response Code | Cause                                                           |
|---------------|-----------------------------------------------------------------|
| 403 Forbidden | The client does not have the required privileges to list users. |

--------------------------------------------------------------------------------

## /users/{user_id}
### GET
Returns the User object with the ID specified by `user_id`.
#### Errors
| Response Code | Cause                                                                     |
|---------------|---------------------------------------------------------------------------|
| 404 Not Found | No user with the requested ID exists.                                     |
| 403 Forbidden | The client does not have the required privileges to view the user's data. |

### PATCH
Updates the User object with the ID specified by `user_id`.
#### Request
A partial User object without the id or createdAt fields.
#### Response
The updated User object.
#### Errors
| Response Code | Cause                                                                     |
|---------------|---------------------------------------------------------------------------|
| 404 Not Found | No user with the requested ID exists.                                     |
| 403 Forbidden | The client does not have the required privileges to edit the user's data. |

### DELETE
Deletes the User with the ID specified by `user_id`.
#### Errors
| Response Code | Cause                                                                |
|---------------|----------------------------------------------------------------------|
| 404 Not Found | No user with the requested ID exists.                                |
| 403 Forbidden | The client does not have the required privileges to delete the user. |

--------------------------------------------------------------------------------

## /users/current
Requests made to this endpoint should be redirected using HTTP status 307 to the user object of the currently authenticated user.
If the client is not authenticated, then the server should respond with a status of 401 - unauthorized.
