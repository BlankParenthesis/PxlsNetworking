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

If the [roles extension](./roles.md) is implemented, the following permissions are added due to this extension:

| Permission                | Purpose                                                                 |
|---------------------------|-------------------------------------------------------------------------|
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
Information on all factions.
Returns a Paginated List of Faction objects.
#### Errors
| Response Code | Cause                                            |
|---------------|--------------------------------------------------|
| 403 Forbidden | The client lacks the permission `factions.list`. |

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
The created Faction object.
#### Errors
| Response Code | Cause                                            |
|---------------|--------------------------------------------------|
| 403 Forbidden | The client lacks the permission `factions.post`. |
| 422 Forbidden | The provided name is invalid.                    |
| 422 Forbidden | The provided tag is invalid.                     |
| 422 Forbidden | The provided color is invalid.                   |

--------------------------------------------------------------------------------

## /factions/{faction_id}
### GET
Information on a specified faction.
Returns the Faction object with the ID matching `faction_id`.
#### Errors
| Response Code | Cause                                           |
|---------------|-------------------------------------------------|
| 403 Forbidden | The client lacks the permission `factions.get`. |
| 404 Not Found | No faction with the requested ID exists.        |

### PATCH
Updates the Faction object with the ID specified by `faction_id`.
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
| Response Code | Cause                                             |
|---------------|---------------------------------------------------|
| 403 Forbidden | The client lacks the permission `factions.patch`. |
| 404 Not Found | No faction with the requested ID exists.          |

### DELETE
Deletes a faction.
#### Errors
| Response Code | Cause                                              |
|---------------|----------------------------------------------------|
| 403 Forbidden | The client lacks the permission `factions.delete`. |
| 404 Not Found | No faction with the requested ID exists.           |

--------------------------------------------------------------------------------

## /factions/{faction_id}/members
### GET
Information on all members of a faction.
Returns a Paginated List of Member objects.
#### Errors
| Response Code | Cause                                                   |
|---------------|---------------------------------------------------------|
| 403 Forbidden | The client lacks the permission `factions.get`.         |
| 404 Not Found | No faction with the requested ID exists.                |
| 403 Forbidden | The client lacks the permission `factions.members.get`. |

### POST
Adds a new member to the Faction with the ID matching `faction_id`.
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
| Response Code | Cause                                                    |
|---------------|----------------------------------------------------------|
| 403 Forbidden | The client lacks the permission `factions.get`.          |
| 404 Not Found | No faction with the requested ID exists.                 |
| 403 Forbidden | The client lacks the permission `factions.members.post`. |
| 404 Not Found | No user with the specified ID exists.                    |
| 409 Conflict  | The user is already a member of the requested faction.   |

--------------------------------------------------------------------------------

## /factions/{faction_id}/members/{member_id}
### GET
Information on a faction member.
Returns the Member specified by `member_id` in the faction specified by `faction_id`.
#### Errors
| Response Code | Cause                                                   |
|---------------|---------------------------------------------------------|
| 403 Forbidden | The client lacks the permission `factions.get`.         |
| 404 Not Found | No faction with the requested ID exists.                |
| 403 Forbidden | The client lacks the permission `factions.members.get`. |
| 404 Not Found | No member with the requested ID exists in the faction.  |

### PATCH
Edits a faction member.
#### Request
A partial Member object without ID (or User) specified.
#### Response
The updated Member object.
#### Errors
| Response Code | Cause                                                     |
|---------------|-----------------------------------------------------------|
| 403 Forbidden | The client lacks the permission `factions.get`.           |
| 404 Not Found | No faction with the requested ID exists.                  |
| 403 Forbidden | The client lacks the permission `factions.members.patch`. |
| 404 Not Found | No member with the requested ID exists in the faction.    |

### DELETE
Removes a faction member.
#### Errors
| Response Code | Cause                                                      |
|---------------|------------------------------------------------------------|
| 403 Forbidden | The client lacks the permission `factions.get`.            |
| 404 Not Found | No faction with the requested ID exists.                   |
| 403 Forbidden | The client lacks the permission `factions.members.delete`. |
| 404 Not Found | No member with the requested ID exists in the faction.     |
