Client Authentication
=====================
Implementing this extension lets clients authenticate their users with the server.
It provides details on how servers can expose available authentication methods for OAuth and direct login.
A server implementing this extension need only provide one method of authentication but could support as many as desired.

It may be beneficial to close registration from particular services for a short time or indefinitely.
To accommodate servers allowing authentication of known users while disallowing new ones, each method can have registration disabled but still present itself as available for authentication.

Authentication options are given by Methods which are defined by the following type:
```typescript
{
	"type": "OAUTH" | "BASIC";
	"name": string;
	"description"?: string;
	"url": string;
	"registration": boolean;
}
```

If the [roles extension](./roles.md) is implemented, the following permissions are added due to this extension:

| Permission     | Purpose                                                |
|----------------|--------------------------------------------------------|
| `roles.post`   | Allows POST requests to `/auth/methods`.               |
| `roles.patch`  | Allows PATCH requests to `/auth/methods/{method_id}`.  |
| `roles.delete` | Allows DELETE requests to `/auth/methods/{method_id}`. |

Server implementations may forgo implementing any of these modifying permissions and do not need to implement the associated modifying endpoints.

--------------------------------------------------------------------------------

## /info
### GET
```typescript
{
	"extensions": ["authentication"];
}
```

--------------------------------------------------------------------------------

## /auth/methods
### GET
A list of all authentication methods.
#### Response
An array of Method References.

### POST
Add a new method which clients can use to to authenticate.
#### Request
A Method object without an ID.
#### Response
A Reference to the created Method object.
#### Errors
| Response Code | Cause                                                                          |
|---------------|--------------------------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to change the authentication methods. |

--------------------------------------------------------------------------------

## {method_uri}
### GET
#### Response
The Method object.

### PATCH
Updates the authentication method.
#### Request
A partial Method object.
#### Response
The updated Method object.
#### Errors
| Response Code | Cause                                                                          |
|---------------|--------------------------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to change the authentication methods. |

### DELETE
Removes the authentication method.
#### Errors
| Response Code | Cause                                                                          |
|---------------|--------------------------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to change the authentication methods. |


**TODO:** "Methods" is not a great name, perhaps come up with a good alternative. Ideally, it would fit in the url and as the object name.