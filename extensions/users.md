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
### Errors
| Response Code | Cause                              |
|---------------|------------------------------------|
| 403 Forbidden | Missing permission `socket.users`. |

--------------------------------------------------------------------------------

## /users
### GET
Returns a Paginated List of User objects.
#### Errors
| Response Code | Cause                            |
|---------------|----------------------------------|
| 403 Forbidden | Missing permission `users.list`. |

--------------------------------------------------------------------------------

## /users/{user_id}
### GET
Returns the User object with the ID specified by `user_id`.
#### Errors
| Response Code | Cause                           |
|---------------|---------------------------------|
| 403 Forbidden | Missing permission `users.get`. |
| 404 Not Found | No such User exists.            |

### PATCH
Updates the User object with the ID specified by `user_id`.
#### Request
A partial User object without the id or createdAt fields.
#### Response
The updated User object.
#### Errors
| Response Code | Cause                             |
|---------------|-----------------------------------|
| 403 Forbidden | Missing permission `users.patch`. |
| 404 Not Found | No such User exists.              |

### DELETE
Deletes the User with the ID specified by `user_id`.
#### Errors
| Response Code | Cause                              |
|---------------|------------------------------------|
| 403 Forbidden | Missing permission `users.delete`. |
| 404 Not Found | No such User exists.               |

--------------------------------------------------------------------------------

## /users/current
Requests made to this endpoint should be redirected using HTTP status 307 to the user object of the currently authenticated user.
If the client is not authenticated, then the server should respond with a status of 401 - unauthorized.
