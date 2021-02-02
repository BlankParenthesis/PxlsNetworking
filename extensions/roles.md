Roles
=====
Implementing this extension provides a way for users distinguish different classes of users using roles.
Roles group users similar to factions (see the [factions extension](./factions.md)): a user can hold multiple roles and many users can hold the same role.
Roles differ from factions in their complexity.
While a faction might have a hierarchy, roles are uniform.

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
}
```

--------------------------------------------------------------------------------

## /info
### GET
```typescript
{
	"extensions": ["roles"];
}
```

--------------------------------------------------------------------------------

## /roles
### GET
Information on all roles.
Returns a Paginated List of Role objects.
#### Errors
| Response Code | Cause                                         |
|---------------|-----------------------------------------------|
| 403 Forbidden | The client lacks the permission `roles.list`. |

--------------------------------------------------------------------------------

## /roles/{role_id}
### GET
Information on a specified role.
Returns the Role object with the id matching `role_id`.
#### Errors
| Response Code | Cause                                        |
|---------------|----------------------------------------------|
| 403 Forbidden | The client lacks the permission `roles.get`. |
| 404 Not Found | No role with the requested ID exists.        |

### PATCH
Updates the Role object with the ID specified by `role_id`.
#### Request
A partial Role object without the ID.
#### Response
The updated Role object.
#### Errors
| Response Code | Cause                                          |
|---------------|------------------------------------------------|
| 403 Forbidden | The client lacks the permission `roles.patch`. |
| 404 Not Found | No role with the requested ID exists.          |

### DELETE
Deletes the Role object with the ID specified by `role_id`.
#### Errors
| Response Code | Cause                                           |
|---------------|-------------------------------------------------|
| 403 Forbidden | The client lacks the permission `roles.delete`. |
| 404 Not Found | No role with the requested ID exists.           |