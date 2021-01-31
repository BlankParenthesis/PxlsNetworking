Roles
=====
Implementing this extension provides a way to distinguish different classes of users and assign different privileges to each class.

This extension does not require that the [users extension](./users.md) is implemented, but if it is this extension modifies the User object to add a list of role names associated with a given user:
```typescript
{
	"roles": string[];
}
```

Role objects are defined by the following type:
```typescript
{
	"id": number | string;
	"name": string;
	"permissions": string[];
	"icon"?: string;
}
```

**DISCUSS:** Assuming the icon is a font-awesome class seems way too implementation specific.
The the best generic way I can think of is specifying an icon src but this would be incompatible with the current implementation.

A list of basic permissions are given here, but other extensions add to this list as needed.
These cover the core protocol: 

| Permission          | Purpose                                                                         |
|---------------------|---------------------------------------------------------------------------------|
| `socket.board`      | Allows connecting to the websocket at `/ws` with `core` in the extensions list. |
| `board.data`        | Allows GET requests to `/board/data` endpoints and `/board/info`.               |
| `board.users`       | Allows GET requests to `/board/users`.                                          |
| `board.pixels.list` | Allows GET requests to `/board/pixels`.                                         |
| `board.pixels.get`  | Allows GET requests to `/board/pixels/{x}/{y}`.                                 |
| `board.pixels.post` | Allows POST requests to `/board/pixels/{x}/{y}`.                                |

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
Information on all roles.
Returns a Paginated List of Role objects.
#### Errors
| Response Code | Cause                                                   |
|---------------|---------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to list roles. |

--------------------------------------------------------------------------------

## /roles/{role_id}
### GET
Information on a specified role.
Returns the Role object with the id matching `role_id`.
#### Errors
| Response Code | Cause                                                      |
|---------------|------------------------------------------------------------|
| 404 Not Found | No role with the requested ID exists.                      |
| 403 Forbidden | The client lacks the required privileges to view the role. |

### PATCH
Updates the Role object with the ID specified by `role_id`.
#### Request
A partial Role object without the ID.
#### Response
The updated Role object.
#### Errors
| Response Code | Cause                                                           |
|---------------|-----------------------------------------------------------------|
| 404 Not Found | No role with the requested ID exists.                           |
| 403 Forbidden | The client does not have the required privileges to edit roles. |

### DELETE
Deletes the Role object with the ID specified by `role_id`.
#### Errors
| Response Code | Cause                                                             |
|---------------|-------------------------------------------------------------------|
| 404 Not Found | No role with the requested ID exists.                             |
| 403 Forbidden | The client does not have the required privileges to delete roles. |