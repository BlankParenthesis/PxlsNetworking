Factions
========
Implementing this extension adds factions which allow users to group themselves into identifiable collectives.

Factions objects are defined by the following type: 
```typescript
{
	"name": string;
	"tag": string;
	"color": number;
	"createdAt": Timestamp;
	"size": number;
}
```

Factions identify users with Member objects which are defined by the following type:
```typescript
{
	"displaysFaction": boolean;
	"approval": {
		"member": boolean;
		"faction": boolean;
	}
	"owner": boolean;
}
```

The approval field can be used by implementations to require approval from a faction after a user has attempted to join before they are formally considered a member.
The field's structure also facilitates users being invited and needing to accept before being formally considered a member.
Implementors might reasonably choose to always return a fully-approved value for this field.
It would be reasonable to automatically remove the member if neither side has approved the member.

If the [users extension](./users.md) is implemented, Member objects will also contain the associated User object:
```typescript
{
	"user": User;
}
```

If the [roles extension](./roles.md) is implemented, the following permissions are added due to this extension:

| Permission                | Purpose                                                                 |
|---------------------------|-------------------------------------------------------------------------|
| `factions.list`           | Allows GET requests to `/factions`.                                     |
| `factions.get`            | Allows GET requests to `/factions/{faction_id}`.                        |
| `factions.post`           | Allows POST requests to `/factions`.                                    |
| `factions.patch`          | Allows PATCH requests to `/factions/{faction_id}`.                      |
| `factions.delete`         | Allows DELETE requests to `/factions/{faction_id}`.                     |
| `factions.members.list`   | Allows GET requests to `/factions/{faction_id}/members`.                |
| `factions.members.get`    | Allows GET requests to `/factions/{faction_id}/members/{member_id}`     |
| `factions.members.post`   | Allows POST requests to `/factions/{faction_id}/members`.               |
| `factions.members.patch`  | Allows PATCH requests to `/factions/{faction_id}/members/{member_id}`.  |
| `factions.members.delete` | Allows DELETE requests to `/factions/{faction_id}/members/{member_id}`. |

*DISCUSS:** Should there be automatically defined permissions for specific cases or should there be a factions roles extension?
Perhaps both options are the correct approach - there should be a sensible way to implement this extension without the other after all.

--------------------------------------------------------------------------------

## /info
### GET
```typescript
{
	"extensions": ["factions"];
}
```

--------------------------------------------------------------------------------

## /factions
### GET
A list of all factions.
#### Response
An array of Faction References.
#### Errors
| Response Code | Cause                                                      |
|---------------|------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to list factions. |

### POST
Create a new Faction.
#### Request
```typescript
{
	"name": string;
	"tag": string;
	"color": number;
}
```
#### Response
A reference to the created Faction object.
#### Errors
| Response Code | Cause                                                        |
|---------------|--------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to create factions. |
| 422 Forbidden | The provided name is invalid.                                |
| 422 Forbidden | The provided tag is invalid.                                 |
| 422 Forbidden | The provided color is invalid.                               |

--------------------------------------------------------------------------------

## {faction_uri}
### GET
#### Response
The Faction object.
#### Errors
| Response Code | Cause                                                         |
|---------------|---------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to view the faction. |

### PATCH
Updates the Faction.
#### Request
```typescript
Partial<{
	"name": string;
	"tag": string;
	"color": number;
}>
```
#### Response
The update Faction object.
#### Errors
| Response Code | Cause                                                           |
|---------------|-----------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to update the faction. |

### DELETE
Deletes the faction.
#### Errors
| Response Code | Cause                                                           |
|---------------|-----------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to delete the faction. |

--------------------------------------------------------------------------------

## {faction_uri}/members
### GET
A list of all members in the faction.
#### Response
An array of Member References.
#### Errors
| Response Code | Cause                                                                 |
|---------------|-----------------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to view the faction members. |

### POST
Add a new member to the Faction.
#### Request
Empty unless the [users extension](./users.md) is implemented in which case, an optional User URI may be specified:
```typescript
{
	"user": string;
}
```
If unspecified, the user added will be the currently authenticated one.
#### Response
A reference to the created Member object.
#### Errors
| Response Code | Cause                                                           |
|---------------|-----------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to join the user.      |
| 404 Not Found | The user URI is invalid.                                        |
| 409 Conflict  | The user is already a member of the requested faction.          |

--------------------------------------------------------------------------------

## {member_uri}
### GET
#### Response
The Member object.
#### Errors
| Response Code | Cause                                                        |
|---------------|--------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to view the member. |

### PATCH
Edit a faction member.
#### Request
A partial Member object.
#### Response
The updated Member object.
#### Errors
| Response Code | Cause                                                        |
|---------------|--------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to edit the member. |

### DELETE
Remove a faction member.
#### Errors
| Response Code | Cause                                                          |
|---------------|----------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to remove the member. |
