Roles
=====
Implementing this extension provides a way for users distinguish different classes of users using roles.
Roles group users similar to factions (see the [factions extension](./factions.md)): a user can hold multiple roles and many users can hold the same role.
Roles differ from factions in their complexity.
While a faction might have a hierarchy, roles are uniform.

Since roles help users distinguish other users, implementing this extension requires implementing the [users extension](./users.md).

Roles also govern which permissions their members have.
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

## Endpoints

### /info
#### GET
##### Response
```typescript
{
	"extensions": ["roles"];
}
```

--------------------------------------------------------------------------------

### /events?subscribe[]={events_list}
#### Server packets
##### RoleCreated
Sent when a role is created.
Set `subscribe[]=roles` to receive.
```typescript
{
	"type": "role-created";
	"role": Reference<Role>;
}
```
##### RoleUpdated
Sent when a role is updated.
Set `subscribe[]=roles` to receive.
```typescript
{
	"type": "role-updated";
	"role": Reference<Role>;
}
```
##### RoleDeleted
Sent when a role is deleted.
Set `subscribe[]=roles` to receive.
```typescript
{
	"type": "role-deleted";
	"role": Url;
}
```
##### UserRolesUpdated
Sent when the roles assigned to a user changes.
Set `subscribe[]=users.roles` to receive, or `subscribe[]=users.current.roles` to receive for the current user.
```typescript
{
	"type": "user-roles-updated";
	"user": Url;
}
```
#### Errors
| Response Code | Cause                                            |
|---------------|--------------------------------------------------|
| 403 Forbidden | Missing permission `events.roles`.               |
| 403 Forbidden | Missing permission `events.users.roles`.         |
| 403 Forbidden | Missing permission `events.users.current.roles`. |

--------------------------------------------------------------------------------

### {user_uri}/roles
#### GET
Lists all roles a user has.
##### Response
A Paginated List of Role objects.
##### Errors
| Response Code | Cause                                                              |
|---------------|--------------------------------------------------------------------|
| 403 Forbidden | Missing permission `users.get` or `users.current.get`.             |
| 404 Forbidden | No such User exists.                                               |
| 403 Forbidden | Missing permission `users.roles.get` or `users.current.roles.get`. |

#### POST
Adds a role to a user.
##### Request
```typescript
{
	"role": string;
}
```
##### Response
A Paginated List of Role objects.
##### Errors
| Response Code | Cause                                                                |
|---------------|----------------------------------------------------------------------|
| 403 Forbidden | Missing permission `users.get` or `users.current.get`.               |
| 403 Forbidden | Missing permission `users.roles.post` or `users.current.roles.post`. |
| 404 Forbidden | No such Role exists.                                                 |

#### DELETE
Removes a role from a user.
##### Response
A Paginated List of Role objects.
##### Request
```typescript
{
	"role": string;
}
```
##### Errors
| Response Code | Cause                                                                    |
|---------------|--------------------------------------------------------------------------|
| 403 Forbidden | Missing permission `users.get` or `users.current.get`.                   |
| 403 Forbidden | Missing permission `users.roles.delete` or `users.current.roles.delete`. |
| 404 Forbidden | No such Role exists.                                                     |

--------------------------------------------------------------------------------

### /roles
#### GET
Lists all roles.
##### Response
A Paginated List of Role References.
##### Errors
| Response Code | Cause                            |
|---------------|----------------------------------|
| 403 Forbidden | Missing permission `roles.list`. |

#### POST
Create a new role.
##### Request
A Role object
##### Response
A Reference to the created Role object.
##### Errors
| Response Code            | Cause                            |
|--------------------------|----------------------------------|
| 403 Forbidden            | Missing permission `roles.post`. |
| 409 Conflict*            | The role already exists.         |
| 422 Unprocessable Entity | Role name is not allowed.        |

> *\*Implementations need not require the role name to be unique, but those that do may use this.*

--------------------------------------------------------------------------------

### {role_uri}
#### GET
Gets the role.
##### Response
The Role object.
##### Errors
| Response Code | Cause                           |
|---------------|---------------------------------|
| 403 Forbidden | Missing permission `roles.get`. |

#### PATCH
Updates the role.
##### Request
A partial Role object.
##### Response
The updated Role object.
##### Errors
| Response Code            | Cause                             |
|--------------------------|-----------------------------------|
| 403 Forbidden            | Missing permission `roles.patch`. |
| 422 Unprocessable Entity | Role name is not allowed.         |

#### DELETE
Deletes the role.
##### Errors
| Response Code | Cause                              |
|---------------|------------------------------------|
| 403 Forbidden | Missing permission `roles.delete`. |
