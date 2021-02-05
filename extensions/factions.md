Factions
========
Implementing this extension adds factions which allow users to group themselves into identifiable collectives.

Factions objects are defined by the following type: 
```typescript
{
	"id": number | string;
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
	"id": number | string;
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

--------------------------------------------------------------------------------

## /info
### GET
#### Response
```typescript
{
	"extensions": ["factions"];
}
```

--------------------------------------------------------------------------------

## /factions
### GET
Lists all factions.
#### Response
A Paginated List of Faction objects.
#### Errors
| Response Code | Cause                               |
|---------------|-------------------------------------|
| 403 Forbidden | Missing permission `factions.list`. |

### POST
Creates a faction.
#### Request
```typescript
{
	"name": string;
	"tag": string;
	"color": number;
}
```
#### Response
The created Faction object.
#### Errors
| Response Code | Cause                               |
|---------------|-------------------------------------|
| 403 Forbidden | Missing permission `factions.post`. |
| 422 Forbidden | Invalid name.                       |
| 422 Forbidden | Invalid tag.                        |
| 422 Forbidden | Invalid color.                      |

--------------------------------------------------------------------------------

## /factions/{faction_id}
### GET
Gets a faction.
#### Response
A Faction object.
#### Errors
| Response Code | Cause                              |
|---------------|------------------------------------|
| 403 Forbidden | Missing permission `factions.get`. |
| 404 Not Found | No such Faction exists.            |

### PATCH
Updates a faction.
#### Request
```typescript
Partial<{
	"name": string;
	"tag": string;
	"color": number;
}>
```
#### Response
The modified Faction.
#### Errors
| Response Code | Cause                                |
|---------------|--------------------------------------|
| 403 Forbidden | Missing permission `factions.patch`. |
| 404 Not Found | No such Faction exists.              |

### DELETE
Deletes a faction.
#### Errors
| Response Code | Cause                                 |
|---------------|---------------------------------------|
| 403 Forbidden | Missing permission `factions.delete`. |
| 404 Not Found | No such Faction exists.               |

--------------------------------------------------------------------------------

## /factions/{faction_id}/members
### GET
Lists all members of a faction.
#### Response
A Paginated List of Member objects.
#### Errors
| Response Code | Cause                                      |
|---------------|--------------------------------------------|
| 403 Forbidden | Missing permission `factions.get`.         |
| 404 Not Found | No such Faction exists.                    |
| 403 Forbidden | Missing permission `factions.members.get`. |

### POST
Adds a new member to a faction.
#### Request
Empty unless the [users extension](./users.md) is implemented in which case, an optional User ID may be specified:
```typescript
{
	"id"?: number | string;
}
```
If unspecified, the user added will be the currently authenticated one.
#### Response
The created Member object.
#### Errors
| Response Code | Cause                                                  |
|---------------|--------------------------------------------------------|
| 403 Forbidden | Missing permission `factions.get`.                     |
| 404 Not Found | No such Faction exists.                                |
| 403 Forbidden | Missing permission `factions.members.post`.            |
| 404 Not Found | No such User exists.                                   |
| 409 Conflict  | The user is already a member of the requested faction. |

--------------------------------------------------------------------------------

## /factions/{faction_id}/members/{member_id}
### GET
Gets a faction member.
#### Response
A Member object.
#### Errors
| Response Code | Cause                                      |
|---------------|--------------------------------------------|
| 403 Forbidden | Missing permission `factions.get`.         |
| 404 Not Found | No such Faction exists.                    |
| 403 Forbidden | Missing permission `factions.members.get`. |
| 404 Not Found | No such Member exists.                     |

### PATCH
Edits a faction member.
#### Request
A partial Member object without ID (or User) specified.
#### Response
The updated Member object.
#### Errors
| Response Code | Cause                                        |
|---------------|----------------------------------------------|
| 403 Forbidden | Missing permission `factions.get`.           |
| 404 Not Found | No such Faction exists.                      |
| 403 Forbidden | Missing permission `factions.members.patch`. |
| 404 Not Found | No such Member exists.                       |

### DELETE
Removes a faction member.
#### Errors
| Response Code | Cause                                         |
|---------------|-----------------------------------------------|
| 403 Forbidden | Missing permission `factions.get`.            |
| 404 Not Found | No such Faction exists.                       |
| 403 Forbidden | Missing permission `factions.members.delete`. |
| 404 Not Found | No such Member exists.                        |
