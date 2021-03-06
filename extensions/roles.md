Roles
=====
Implementing this extension provides a way for users distinguish different classes of users using roles.
Roles group users similar to factions (see the [factions extension](./factions.md)): a user can hold multiple roles and many users can hold the same role.
Roles differ from factions in their complexity.
While a faction might have a hierarchy, roles are uniform.

Since roles help users distinguish other users, implementing this extension requires implementing the [users extension](./users.md).

Roles also govern which permissions its members have.
*Implementations lose the ability to arbitrarily decide a users permissions set by implementing this extensions.*
Instead, a user's permissions are determined entirely based on roles.
Each role defines a set of permission keys and a user's permissions keys should be the union of the permissions for all roles they hold.
Granting a role the permission to edit roles *effectively grants all permissions to that role.*

Role objects are defined by the following type:
```typescript
{
	"id": number | string;
	"name": string;
	"icon"?: string;
	"permissions": string[];
}
```

--------------------------------------------------------------------------------

## /info
### GET
#### Response
```typescript
{
	"extensions": ["roles"];
}
```

--------------------------------------------------------------------------------

## /users/{user_id}/roles
### GET
Lists all roles a user has.
#### Response
A Paginated List of Role objects.
#### Errors
| Response Code | Cause                                  |
|---------------|----------------------------------------|
| 403 Forbidden | Missing permission `users.get`.        |
| 404 Forbidden | No such User exists.                   |
| 403 Forbidden | Missing permission `users.roles.post`. |

### POST
Adds a role to a user.
#### Request
```typescript
{
	"role": number | string;
}
```
#### Response
A Paginated List of Role objects.
#### Errors
| Response Code | Cause                                  |
|---------------|----------------------------------------|
| 403 Forbidden | Missing permission `users.get`.        |
| 404 Forbidden | No such User exists.                   |
| 403 Forbidden | Missing permission `users.roles.post`. |

--------------------------------------------------------------------------------

## /users/{user_id}/roles/{role_id}
### DELETE
Removes a role from a user.
#### Errors
| Response Code | Cause                                    |
|---------------|------------------------------------------|
| 403 Forbidden | Missing permission `users.get`.          |
| 404 Forbidden | No such User exists.                     |
| 403 Forbidden | Missing permission `users.roles.delete`. |
| 404 Forbidden | No such Role exists.                     |

--------------------------------------------------------------------------------

## /roles
### GET
Lists all roles.
#### Response
A Paginated List of Role objects.
#### Errors
| Response Code | Cause                            |
|---------------|----------------------------------|
| 403 Forbidden | Missing permission `roles.list`. |

### POST
Creates a role.
#### Request
A Role object without an ID.
#### Response
The created Role object.
#### Errors
| Response Code | Cause                            |
|---------------|----------------------------------|
| 403 Forbidden | Missing permission `roles.post`. |

--------------------------------------------------------------------------------

## /roles/{role_id}
### GET
Gets a role.
#### Response
A Role object.
#### Errors
| Response Code | Cause                           |
|---------------|---------------------------------|
| 403 Forbidden | Missing permission `roles.get`. |
| 404 Not Found | No such Role exists.            |

### PATCH
Updates a role.
#### Request
A partial Role object without the ID.
#### Response
The updated Role object.
#### Errors
| Response Code | Cause                             |
|---------------|-----------------------------------|
| 403 Forbidden | Missing permission `roles.patch`. |
| 404 Not Found | No such Role exists.              |

### DELETE
Deletes a role.
#### Errors
| Response Code | Cause                              |
|---------------|------------------------------------|
| 403 Forbidden | Missing permission `roles.delete`. |
| 404 Not Found | No such Role exists.               |