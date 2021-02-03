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
	"type": "OAUTH" | "FORM" | "OAUTH_LOCAL";
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

Methods that use the "OAUTH_LOCAL" type function similar to "OAUTH" except that the client can guarantee that the final redirect URL will be `localhost:59415`.
Thanks to this, a client set up a webserver on that address and listen for the response and forward it back to the server at `/auth/oauth/local`.
The client can expect the server's response to contain the Set-Cookie header similarly to the "FORM" authentication method.
Clients should send the indicated cookies back to the server for any future requests and should expect to be authenticated when they do so.

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
Information about how and where clients can authenticate.
#### Response
A Paginated List of Methods.

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
#### Response
A Method object.

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