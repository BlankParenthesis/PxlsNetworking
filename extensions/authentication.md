Client Authentication
=====================
Implementing this extension lets clients authenticate their users with the server.
It provides endpoints indicating where and how clients should authenticate and continue to appear authenticated.

Clients are expected to obtain an OAuth 2.0 token and send this in the authorization header as per [rfc6750](https://datatracker.ietf.org/doc/html/rfc6750#section-2.1).

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

## /auth
### GET
Gets information about where and how clients can obtain authentication tokens.
#### Response
```typescript
{
	"issuer": string;
	"client_id"?: string;
}
```
`issuer` is the location of the OpenID Connect issuer.
Clients can append `/.well-known/openid-configuration` to this location to perform OIDC discovery and determine the required endpoints for authentication.

Clients should use the OAuth 2.0 PKCE method described in [rfc7636](https://datatracker.ietf.org/doc/html/rfc7636) for initiating the login flow and retrieving the token.
If `client_id` is present, it is public and supports any redirect URI with no required secret.
Otherwise, clients will need to use a known client id (and probably secret).
Knowing this id and secret implies a pre-existing relationship between the client and server, making complete interchangeability difficult in these cases.

--------------------------------------------------------------------------------

**TODO:** scopes