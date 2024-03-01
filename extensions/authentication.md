Client Authentication
=====================
Implementing this extension lets clients authenticate their users with the server.
It provides endpoints indicating where and how clients should authenticate and continue to appear authenticated.

Clients are expected to obtain an OAuth 2.0 token and send this in the authorization header as per [rfc6750](https://datatracker.ietf.org/doc/html/rfc6750#section-2.1).

--------------------------------------------------------------------------------

## Endpoints

### /info
#### GET
##### Response
```typescript
{
	"extensions": ["authentication"];
}
```

--------------------------------------------------------------------------------

### /auth
#### GET
Gets information about where and how clients can obtain authentication tokens.
##### Response
```typescript
{
	"issuer": string;
	"client_id"?: string;
}
```
`issuer` is the location of the OpenID Connect issuer.
Clients can append `/.well-known/openid-configuration` to this location to perform OIDC discovery and determine the required endpoints for authentication.

Clients should use the OAuth 2.0 PKCE method described in [rfc7636](https://datatracker.ietf.org/doc/html/rfc7636) for initiating the login flow and retrieving the token.
Clients may then refresh this token as required.
If `client_id` is present and not null, it should be public and support any redirect URI with no required secret.
Otherwise, clients will need to use a known public client ID or a private client ID and secret.
Knowing this ID and secret implies a pre-existing relationship between the client and server, making complete interchangeability difficult in these cases.

--------------------------------------------------------------------------------

### /socket?extensions[]=authentication
*NOTE: The definitions here also apply to board sockets*
#### Client packets
Unlike regular HTTP requests, websocket connections have a significant duration.
Credentials are unlikely to expire over the course of a regular HTTP request.
This allows regular requests to use headers for authentication.
Websocket connections are likely to last long enough that credentials will expire.
This makes HTTP headers an inadequate method of authentication.

Clients wishing to authenticate a websocket connection are required to send authenticate packets before the current authentication expires.
Servers should not send any packets until the client's authentication is verified.
Servers should keep connections open for at least 5 seconds to allow clients time to send an initial authenticate packet.
##### Authenticate
Set authentication state.
```typescript
{
	"type": "authenticate";
	"token"?: string;
}
```
If `token` is not provided, the client should be treated as unauthenticated and should no longer send authenticate packets.

#### Errors
With the sole exception of `socket.authenticate`, socket requests should not check user permissions until after the client sends the first Authenticate packet if using this extension.
*NOTE: Since clients are authenticated after connection, the initial request is tested against the unauthenticated user permissions.*
| Response Code            | Cause                                     |
|--------------------------|-------------------------------------------|
| 403 Forbidden            | Missing permission `socket.authenticate`. |
#### Websocket Errors
| Close Code | Cause                                       |
|------------|---------------------------------------------|
| 4000       | Client failed to send Authenticate in time. |
| 4001       | Missing permission.                         |
| 4002       | Invalid token.                              |

*NOTE: 4001 should be sent to indicate the client lacks a permission after a successful authentication.*