Users
=====
Implementing this extension attaches a unique persistent profile to every authenticated user.
Consequently, implementing this extension requires implementing the [authentication extension](./authentication.md).

User objects are represented with the following type:
```typescript
{
	"name": string;
	"created_at": Timestamp;
}
```

Placement objects gain an additional field due to this extension:
```typescript
{
	"user"?: Reference<User>;
}
```

--------------------------------------------------------------------------------

## Endpoints

### /info
#### GET
##### Response
```typescript
{
	"extensions": ["users"];
}
```

--------------------------------------------------------------------------------

### /socket?extensions[]=users
#### Server packets
##### UserUpdate
The client user has changed.
```typescript
{
	"type": "user-updated";
	"user": Reference<User>;
}
```
#### Errors
| Response Code | Cause                              |
|---------------|------------------------------------|
| 403 Forbidden | Missing permission `socket.users`. |

--------------------------------------------------------------------------------

### /users
#### GET
Lists all users.
##### Response
A Paginated List of User References.
##### Errors
| Response Code | Cause                            |
|---------------|----------------------------------|
| 403 Forbidden | Missing permission `users.list`. |

--------------------------------------------------------------------------------

### {user_uri}
#### GET
Gets a user.
##### Response
The User object.
##### Errors
| Response Code | Cause                                                  |
|---------------|--------------------------------------------------------|
| 403 Forbidden | Missing permission `users.get` or `users.current.get`. |

#### PATCH
Updates a user.
##### Request
A partial User object without created_at field.
##### Response
The updated User object.
##### Errors
| Response Code            | Cause                                                      |
|--------------------------|------------------------------------------------------------|
| 403 Forbidden            | Missing permission `users.patch` or `users.current.patch`. |
| 422 Unprocessable Entity | New username is not allowed.                               |

#### DELETE
Deletes a user.
##### Errors
| Response Code | Cause                                                        |
|---------------|--------------------------------------------------------------|
| 403 Forbidden | Missing permission `users.delete` or `users.current.delete`. |
| 404 Not Found | No such User exists.                                         |

--------------------------------------------------------------------------------

### /users/current
Requests made to this endpoint should be redirected using HTTP status 307 to the user object of the currently authenticated user.

If any sub-endpoints are called on this, they should likewise be redirected, keeping the path.
##### Errors
| Response Code    | Cause                                   |
|------------------|-----------------------------------------|
| 401 Unauthorized | Not authenticated.                      |
| 403 Forbidden    | Missing permission `users.current.get`. |
