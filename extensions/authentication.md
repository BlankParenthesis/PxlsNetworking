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
	"type": "OAUTH" | "FORM";
	"name": string;
	"description"?: string;
	"url": string;
	"registration": boolean;
}
```
Methods that use the "OAUTH" type of authentication provide a URL which will initiate an OAuth flow.
Client implementations have no control over this flow.
Web clients served from the same origin as the server can simply redirect the page to the chosen flow and can expect to eventually be revisited at the completion of the flow with authentication in place.
Other clients cannot expect this and should not use this type of method.

Methods that use the "FORM" type of authentication provide a URL to which a client can POST a `username` and `password`.
Responses from the provided URL should return a response with the Set-Cookie header when successful.
Clients should send the indicated cookies back to the server for any future requests and should expect to be authenticated when they do so.

--------------------------------------------------------------------------------

## /info
### GET
#### Response
```typescript
{
	"extensions": ["authentication"];
}
```

--------------------------------------------------------------------------------

## /auth/methods
### GET
Lists authentication methods.
#### Response
A Paginated List of Method References.

### POST
Creates an authentication method.
#### Request
A Method object without an ID.
#### Response
A Reference to the created Method object.
#### Errors
| Response Code | Cause                           |
|---------------|---------------------------------|
| 403 Forbidden | Missing permission `auth.post`. |

--------------------------------------------------------------------------------

## {method_uri}
### GET
Gets an authentication method.
#### Response
The Method object.

### PATCH
Updates the authentication method.
#### Request
A partial Method object.
#### Response
The updated Method object.
#### Errors
| Response Code | Cause                          |
|---------------|--------------------------------|
| 403 Forbidden | Missing permission `auth.get`. |

### DELETE
Deletes the authentication method.
#### Errors
| Response Code | Cause                             |
|---------------|-----------------------------------|
| 403 Forbidden | Missing permission `auth.delete`. |


**TODO:** "Methods" is not a great name, perhaps come up with a good alternative. Ideally, it would fit in the url and as the object name.