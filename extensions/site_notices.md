Site Notices
============
Implementing this extension provides a way for site administrators to post long-term notices which inform users of something.

For per-board notices, see the [board notices extension](./board_notices.md).
To help differentiate the two, many of the names pertaining to notices here have been prefixed with "site" despite existing at a root scope.

Notice objects represent notices and are defined by the following type:
```typescript
{
	"id": number | string;
	"title": string;
	"content": string;
	"created_at": Timestamp;
	"expires_at": Timestamp;
}
```

If the [users extension](./users.md) is implemented, Notice objects gain an optional author field:
```typescript
{
	"author"?: User;
}
```

--------------------------------------------------------------------------------

## /info
### GET
#### Response
```typescript
{
	"extensions": ["site_notices"];
}
```

--------------------------------------------------------------------------------

## /ws?extensions[]=site_notices
### Server packets
#### SiteNoticeCreated
There is a new site notice.
```typescript
{
	"type": "site-notice-created";
	"notice": Notice;
}
```
#### SiteNoticeUpdated
A site notice has been updated.
```typescript
{
	"type": "site-notice-updated";
	"notice": Notice;
}
```
#### SiteNoticetRemoved
A site notice has been removed.
```typescript
{
	"type": "site-notice-removed";
	"id": number | string;
}
```
### Errors
| Response Code | Cause                                     |
|---------------|-------------------------------------------|
| 403 Forbidden | Missing permission `socket.site.notices`. |

--------------------------------------------------------------------------------

## /notice
### GET
Lists all site notice.
#### Response
A Paginated List of Notice References.
#### Errors
| Response Code | Cause                                   |
|---------------|-----------------------------------------|
| 403 Forbidden | Missing permission `site.notices.list`. |

### POST
Creates a site notice.
#### Request
A Notice object without an ID (or author if the [users extension](./users.md) is implemented).
#### Response
A reference to the created Notice.
#### Errors
| Response Code | Cause                                   |
|---------------|-----------------------------------------|
| 403 Forbidden | Missing permission `site.notices.post`. |

--------------------------------------------------------------------------------

## {notices_uri}
### GET
Gets a site notice.
#### Response
A Notice object.
#### Errors
| Response Code | Cause                                  |
|---------------|----------------------------------------|
| 403 Forbidden | Missing permission `site.notices.get`. |
| 404 Not Found | No such Notice exists.                 |

### PATCH
Updates a site notice.
#### Request
A partial Notice object without an ID (or author if the [users extension](./users.md) is implemented).
#### Response
The updated Notice object.
#### Errors
| Response Code | Cause                                    |
|---------------|------------------------------------------|
| 403 Forbidden | Missing permission `site.notices.patch`. |
| 404 Not Found | No such Notice exists.                   |

### DELETE
Deletes a site notice.
#### Errors
| Response Code | Cause                                     |
|---------------|-------------------------------------------|
| 403 Forbidden | Missing permission `site.notices.delete`. |
| 404 Not Found | No such Notice exists.                    |
