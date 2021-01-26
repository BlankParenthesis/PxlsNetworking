Roles
=====
Implementing this extension provides a way to distinguish different classes of users and assign different privileges to each class.

This extension does not require that the [users extension](./users.md) is implemented, but if it is this extension modifies the User object to add a list of Role References for the roles associated with a given user:
```typescript
{
	"roles": Reference<Role>[];
}
```

Role objects are defined by the following type:
```typescript
{
	"name": string;
	"permissions": string[];
	"icon"?: string;
}
```

**DISCUSS:** Assuming the icon is a font-awesome class seems way too implementation specific.
The the best generic way I can think of is specifying an icon src but this would be incompatible with the current implementation.

A list of basic permissions are given here, but other extensions add to this list as needed.
These cover the core protocol: 

| Permission          | Purpose                                                           |
|---------------------|-------------------------------------------------------------------|
| `board.data`        | Allows GET requests to `/board/data` endpoints and `/board/info`. |
| `board.users`       | Allows GET requests to `/board/users`.                            |
| `board.socket`      | Allows connecting to the websocket at `/board/ws`.                |
| `board.pixels.get`  | Allows GET requests to `/board/pixels/{x}/{y}`.                   |
| `board.pixels.post` | Allows POST requests to `/board/pixels/{x}/{y}`.                  |

These permissions cover the extra functionality added by this extension: 

| Permission      | Purpose                                         |
|-----------------|-------------------------------------------------|
| `roles.list`    | Allows GET requests to `/roles`.                |
| `roles.get`     | Allows GET requests to `/roles/{role_name}`.    |
| `roles.post`    | Allows POST requests to `/roles`.               |
| `roles.patch`   | Allows PATCH requests to `/roles/{role_name}`.  |
| `roles.delete`  | Allows DELETE requests to `/roles/{role_name}`. |

Server implementations may forgo implementing any of these modifying permissions and do not need to implement the associated modifying endpoints.

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
A list of all roles.
#### Response
An array of Role References.
#### Errors
| Response Code | Cause                                                   |
|---------------|---------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to list roles. |

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
#### Response
The Role object.
#### Errors
| Response Code | Cause                                                      |
|---------------|------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to view the role. |

### PATCH
Updates the Role object.
#### Request
A partial Role object.
#### Response
The updated Role object.
#### Errors
| Response Code | Cause                                                           |
|---------------|-----------------------------------------------------------------|
| 403 Forbidden | The client does not have the required privileges to edit roles. |

### DELETE
Deletes the Role object.
#### Errors
| Response Code | Cause                                                             |
|---------------|-------------------------------------------------------------------|
| 403 Forbidden | The client does not have the required privileges to delete roles. |