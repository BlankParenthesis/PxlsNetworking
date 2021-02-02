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
	"id": number | string;
	"type": "OAUTH" | "BASIC";
	"name": string;
	"description"?: string;
	"url": string;
	"registration": boolean;
}
```

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
Information about how and where clients can authenticate.
Returns a Paginated List of Methods.

### POST
Add a new method which clients can use to to authenticate.
#### Request
A Method object without an ID.
#### Response
The created Method object.
#### Errors
| Response Code | Cause                           |
|---------------|---------------------------------|
| 403 Forbidden | Missing permission `auth.post`. |

--------------------------------------------------------------------------------

## /auth/methods/{method_id}
### GET
Information on the method with ID equal to `method_id`.
Returns a Method object.

### PATCH
Updates an authentication method.
#### Request
A partial Method object without an ID.
#### Response
The modified Method object.
#### Errors
| Response Code | Cause                          |
|---------------|--------------------------------|
| 404 Not Found | No such Method exists.         |
| 403 Forbidden | Missing permission `auth.get`. |

### DELETE
Removes an authentication method.
#### Errors
| Response Code | Cause                             |
|---------------|-----------------------------------|
| 404 Not Found | No such Method exists.            |
| 403 Forbidden | Missing permission `auth.delete`. |


**TODO:** "Methods" is not a great name, perhaps come up with a good alternative. Ideally, it would fit in the url and as the object name.