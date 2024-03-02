Factions
========
Implementing this extension adds factions which allow users to group themselves into identifiable collectives.

Factions objects are defined by the following type: 
```typescript
{
	"name": string;
	"created_at": Timestamp;
	"size": number;
}
```

Factions identify users with Member objects which are defined by the following type:
```typescript
{
	"join_intent": {
		"member": boolean;
		"faction": boolean;
	}
	"owner": boolean;
}
```

The join_intent field can optionally be used by implementations to do any of the following:
- require approval from a faction after a user has attempted to join before they are formally considered a member
- let a faction invite a user, which may be the only way that user can join that faction
Implementors might reasonably choose to always return a fully-approved value for this field and skip these processes.

If both sub-fields are false, the user should no longer be considered a member.

If the [users extension](./users.md) is implemented, Member objects will also contain the associated User object:
```typescript
{
	"user": Reference<User>;
}
```

--------------------------------------------------------------------------------

## Endpoints

### /info
#### GET
##### Response
```typescript
{
	"extensions": ["factions"];
}
```

--------------------------------------------------------------------------------

### /factions
#### GET
Lists all factions.
##### Response
A Paginated List of Faction References.
##### Errors
| Response Code | Cause                               |
|---------------|-------------------------------------|
| 403 Forbidden | Missing permission `factions.list`. |

#### POST
Creates a faction.
##### Request
A Faction object without created_at or size.
##### Response
A reference to the created Faction object.
##### Errors
| Response Code | Cause                               |
|---------------|-------------------------------------|
| 403 Forbidden | Missing permission `factions.post`. |
| 422 Forbidden | Invalid name.                       |
| 422 Forbidden | Invalid tag.                        |
| 422 Forbidden | Invalid color.                      |

--------------------------------------------------------------------------------

### {faction_uri}
#### GET
##### Response
The Faction object.
##### Errors
| Response Code | Cause                              |
|---------------|------------------------------------|
| 403 Forbidden | Missing permission `factions.get`. |

#### PATCH
Updates the Faction.
##### Request
A partial Faction object without created_at or size.
##### Response
The update Faction object.
##### Errors
| Response Code | Cause                                |
|---------------|--------------------------------------|
| 403 Forbidden | Missing permission `factions.patch`. |

#### DELETE
Deletes the faction.
##### Errors
| Response Code | Cause                                 |
|---------------|---------------------------------------|
| 403 Forbidden | Missing permission `factions.delete`. |

--------------------------------------------------------------------------------

### {faction_uri}/members
#### GET
Lists all members of a faction.
##### Response
A Paginated List of Member objects.
##### Errors
| Response Code | Cause                                      |
|---------------|--------------------------------------------|
| 403 Forbidden | Missing permission `factions.get`.         |
| 403 Forbidden | Missing permission `factions.members.get`. |

#### POST
Adds a new member to a faction.
##### Request
Empty unless the [users extension](./users.md) is implemented in which case, an optional User URI may be specified:
```typescript
{
	"user": string;
}
```
If unspecified, the user added will be the currently authenticated one.
##### Response
A reference to the created Member object.
##### Errors
| Response Code | Cause                                                  |
|---------------|--------------------------------------------------------|
| 403 Forbidden | Server does not support the action (joining/inviting). |
| 403 Forbidden | Missing permission `factions.get`.                     |
| 403 Forbidden | Missing permission `factions.members.post`.            |
| 404 Not Found | The user URI is invalid.                               |
| 409 Conflict  | The user is already a member of the requested faction. |

--------------------------------------------------------------------------------

### {member_uri}
#### GET
Gets a faction member.
##### Response
A Member object.
##### Errors
| Response Code | Cause                                      |
|---------------|--------------------------------------------|
| 403 Forbidden | Missing permission `factions.get`.         |
| 403 Forbidden | Missing permission `factions.members.get`. |

#### PATCH
Edit a faction member.
##### Request
A partial Member object.
##### Response
The updated Member object.
##### Errors
| Response Code | Cause                                        |
|---------------|----------------------------------------------|
| 403 Forbidden | Missing permission `factions.get`.           |
| 403 Forbidden | Missing permission `factions.members.patch`. |

#### DELETE
Remove a faction member.
##### Errors
| Response Code | Cause                                         |
|---------------|-----------------------------------------------|
| 403 Forbidden | Missing permission `factions.get`.            |
| 403 Forbidden | Missing permission `factions.members.delete`. |
