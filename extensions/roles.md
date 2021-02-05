Roles
=====
Implementing this extension provides a way for users distinguish different classes of users using roles.
Roles group users similar to factions (see the [factions extension](./factions.md)): a user can hold multiple roles and many users can hold the same role.
Roles differ from factions in their complexity.
While a faction might have a hierarchy, roles are uniform.

Roles also govern which permissions its members have.
*Implementations lose the ability to arbitrarily decide a users permissions set by implementing this extensions.*
Instead, a user's permissions are determined entirely based on roles.
Each role defines a set of permission keys and a user's permissions keys should be the union of the permissions for all roles they hold.
Granting a role the permission to edit roles *effectively grants all permissions to that role.*

Since roles help users distinguish other users, implementing this extension requires implementing the [users extension](./users.md).
User objects are given a new field to represent the roles associated with a user:
```typescript
{
	"roles": Array<number | string>;
}
```
This field contains a list of Role object IDs.

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

## /roles
### GET
Information on all roles.
#### Response
A Paginated List of Role objects.
#### Errors
| Response Code | Cause                            |
|---------------|----------------------------------|
| 403 Forbidden | Missing permission `roles.list`. |

--------------------------------------------------------------------------------

## /roles/{role_id}
### GET
Information on a specified role.
#### Response
A Role object.
#### Errors
| Response Code | Cause                           |
|---------------|---------------------------------|
| 403 Forbidden | Missing permission `roles.get`. |
| 404 Not Found | No such Role exists.            |

### PATCH
Updates the specified role.
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
Deletes the specified role.
#### Errors
| Response Code | Cause                              |
|---------------|------------------------------------|
| 403 Forbidden | Missing permission `roles.delete`. |
| 404 Not Found | No such Role exists.               |