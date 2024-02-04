Roles
=====
Implementing this extension provides a way for users distinguish different classes of users using roles.
Roles group users similar to factions (see the [factions extension](./factions.md)): a user can hold multiple roles and many users can hold the same role.
Roles differ from factions in their complexity.
While a faction might have a hierarchy, roles are uniform.

Since roles help users distinguish other users, implementing this extension requires implementing the [users extension](./users.md).

Roles also govern which permissions its members have.
*By implementing this extension, implementations lose the ability to arbitrarily decide a users permissions.*
Instead, a user's permissions are determined entirely based on roles.
Each role defines a set of permissions and a user's permissions should be the union of the permissions for all roles they hold.
Granting a role the permission to edit roles *effectively grants all permissions to that role.*

Role objects are defined by the following type:
```typescript
{
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

## {user_uri}/roles
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
	"role": string;
}
```
#### Response
A Paginated List of Role objects.
#### Errors
| Response Code | Cause                                  |
|---------------|----------------------------------------|
| 403 Forbidden | Missing permission `users.get`.        |
| 403 Forbidden | Missing permission `users.roles.post`. |
| 404 Forbidden | No such Role exists.                   |

### DELETE
Removes a role from a user.
#### Request
```typescript
{
	"role": string;
}
```
#### Errors
| Response Code | Cause                                    |
|---------------|------------------------------------------|
| 403 Forbidden | Missing permission `users.get`.          |
| 403 Forbidden | Missing permission `users.roles.delete`. |
| 404 Forbidden | No such Role exists.                     |

--------------------------------------------------------------------------------

## /roles
### GET
Lists all roles.
#### Response
A Paginated List of Role References.
#### Errors
| Response Code | Cause                            |
|---------------|----------------------------------|
| 403 Forbidden | Missing permission `roles.list`. |

### POST
Creates a role.
#### Request
A Role object.
#### Response
The created Role object.
#### Errors
| Response Code | Cause                            |
|---------------|----------------------------------|
| 403 Forbidden | Missing permission `roles.post`. |

### POST
Create a new role.
#### Request
A role object
#### Response
A reference to the created Role object.
#### Errors
| Response Code | Cause                                                     |
|---------------|-----------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to create roles. |

--------------------------------------------------------------------------------

## {role_uri}
### GET
Gets the role.
#### Response
The Role object.
#### Errors
| Response Code | Cause                           |
|---------------|---------------------------------|
| 403 Forbidden | Missing permission `roles.get`. |

### PATCH
Updates the role.
#### Request
A partial Role object.
#### Response
The updated Role object.
#### Errors
| Response Code | Cause                             |
|---------------|-----------------------------------|
| 403 Forbidden | Missing permission `roles.patch`. |

### DELETE
Deletes the role.
#### Errors
| Response Code | Cause                              |
|---------------|------------------------------------|
| 403 Forbidden | Missing permission `roles.delete`. |
