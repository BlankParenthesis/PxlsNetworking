Site Notices
============
Implementing this extension provides a way for site administrators to post long-term notices which inform users of something.

For per-board notices, see the [board notices extension](./board_notices.md).
To help differentiate the two, many of the names pertaining to notices here have been prefixed with "site" despite existing at a root scope.

Notice objects represent notices and are defined by the following type:
```typescript
{
	"title": string;
	"content": string;
	"created_at": Timestamp;
	"expires_at"?: Timestamp;
}
```

If the [users extension](./users.md) is implemented, Notice objects gain an optional author field:
```typescript
{
	"author"?: Reference<User>;
}
```

--------------------------------------------------------------------------------

## Endpoints

### /info
#### GET
##### Response
```typescript
{
	"extensions": ["site_notices"];
}
```

--------------------------------------------------------------------------------

### /events?subscribe[]={events_list}
#### Server packets
##### SiteNoticeCreated
Sent when a site notice is created.
Set `subscribe[]=notices` to receive.
```typescript
{
	"type": "site-notice-created";
	"notice": Reference<Notice>;
}
```
##### SiteNoticeUpdated
Sent when a site notice is updated.
Set `subscribe[]=notices` to receive.
```typescript
{
	"type": "site-notice-updated";
	"notice": Reference<Notice>;
}
```
##### SiteNoticetRemoved
Sent when a site notice is removed.
Set `subscribe[]=notices` to receive.
```typescript
{
	"type": "site-notice-removed";
	"notice": Reference<Notice>;
}
```
#### Errors
| Response Code | Cause                                |
|---------------|--------------------------------------|
| 403 Forbidden | Missing permission `events.notices`. |

--------------------------------------------------------------------------------

### /notices
#### GET
Lists all site notice.
##### Response
A Paginated List of Notice References.
##### Errors
| Response Code | Cause                              |
|---------------|------------------------------------|
| 403 Forbidden | Missing permission `notices.list`. |

#### POST
Creates a site notice.
##### Request
A Notice object.
*NOTE: the author field from the [users extension](./users.md) should not be included here*
##### Response
A reference to the created Notice.
##### Errors
| Response Code | Cause                              |
|---------------|------------------------------------|
| 403 Forbidden | Missing permission `notices.post`. |

--------------------------------------------------------------------------------

### {notices_uri}
#### GET
Gets a site notice.
##### Response
A Notice object.
##### Errors
| Response Code | Cause                             |
|---------------|-----------------------------------|
| 403 Forbidden | Missing permission `notices.get`. |
| 404 Not Found | No such Notice exists.            |

#### PATCH
Updates a site notice.
##### Request
A partial Notice object.
*NOTE: the author field from the [users extension](./users.md) should not be included here*
##### Response
The updated Notice object.
##### Errors
| Response Code | Cause                               |
|---------------|-------------------------------------|
| 403 Forbidden | Missing permission `notices.patch`. |
| 404 Not Found | No such Notice exists.              |

#### DELETE
Deletes a site notice.
##### Errors
| Response Code | Cause                                |
|---------------|--------------------------------------|
| 403 Forbidden | Missing permission `notices.delete`. |
| 404 Not Found | No such Notice exists.               |
