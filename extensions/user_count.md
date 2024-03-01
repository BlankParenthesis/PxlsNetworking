User Count
==========
Implementing this extension allows clients to fetch the number of recently active users.
This is up to the server implementation to determine, but a reasonable definition is the number of place events from unique users that have occurred within the `idle_timeout`.

This count is commonly used in calculating cooldown times and the server should expose the same values used internally if this is the case.

--------------------------------------------------------------------------------

## Endpoints

### /info
#### GET
##### Response
```typescript
{
	"extensions": ["user_count"];
}
```

--------------------------------------------------------------------------------

### {board_uri}/users
#### GET
Gets the active user count.
##### Response
```typescript
{
	"active": number;
	"idle_timeout": number;
}
```
`idle_timeout` is in seconds.
##### Errors
| Response Code | Cause                              |
|---------------|------------------------------------|
| 403 Forbidden | Missing permission `boards.users`. |