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
#### Response
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
Lists all users.
#### Response
A Paginated List of User objects.
#### Errors
| Response Code | Cause                            |
|---------------|----------------------------------|
| 403 Forbidden | Missing permission `users.list`. |

--------------------------------------------------------------------------------

## /users/{user_id}
### GET
Gets a user.
#### Response
A User object.
#### Errors
| Response Code | Cause                                                  |
|---------------|--------------------------------------------------------|
| 403 Forbidden | Missing permission `users.get` or `users.current.get`. |
| 404 Not Found | No such User exists.                                   |

### PATCH
Updates a user.
#### Request
A partial User object without the id or createdAt fields.
#### Response
The updated User object.
#### Errors
| Response Code | Cause                                                      |
|---------------|------------------------------------------------------------|
| 403 Forbidden | Missing permission `users.patch` or `users.current.patch`. |
| 404 Not Found | No such User exists.                                       |

### DELETE
Deletes a user.
#### Errors
| Response Code | Cause                                                        |
|---------------|--------------------------------------------------------------|
| 403 Forbidden | Missing permission `users.delete` or `users.current.delete`. |
| 404 Not Found | No such User exists.                                         |

--------------------------------------------------------------------------------

## /users/current
Requests made to this endpoint should be redirected using HTTP status 307 to the user object of the currently authenticated user.
If the client is not authenticated, then the server should respond with a status of 401 - unauthorized.
