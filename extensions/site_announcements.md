Site Announcements
==================
Implementing this extension provides a way for site administrators to make long-term announcements which inform users of something.

For board-related announcements, see the [board announcements extension](./board_announcements.md).
To help differentiate the two, many of the names pertaining to announcements here have been prefixed with "site" despite existing at a root scope.

Announcement objects represent announcements and are defined by the following type:
```typescript
{
	"id": number | string;
	"title": string;
	"content": string;
	"created_at": Timestamp;
	"expires_at": Timestamp;
}
```

If the [users extension](./users.md) is implemented, Announcement objects gain an optional author field:
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
	"extensions": ["site_announcements"];
}
```

--------------------------------------------------------------------------------

## /ws?extensions[]=site_announcements
### Server packets
#### SiteAnnouncementCreated
There is a new site announcement.
```typescript
{
	"type": "site-announcement-created";
	"announcement": Announcement;
}
```
#### SiteAnnouncementUpdated
A site announcement has been updated.
```typescript
{
	"type": "site-announcement-updated";
	"announcement": Announcement;
}
```
#### SiteAnnouncementRemoved
A site announcement has been removed.
```typescript
{
	"type": "site-announcement-removed";
	"id": number | string;
}
```
### Errors
| Response Code | Cause                                      |
|---------------|--------------------------------------------|
| 403 Forbidden | Missing permission `socket.site.announcements`. |

--------------------------------------------------------------------------------

## /announcements
### GET
Lists all site announcements.
#### Response
A Paginated List of Announcement References.
#### Errors
| Response Code | Cause                                          |
|---------------|------------------------------------------------|
| 403 Forbidden | Missing permission `site.announcements.list`. |

### POST
Creates a site announcement.
#### Request
An Announcement object without an ID (or author if the [users extension](./users.md) is implemented).
#### Response
A reference to the created Announcement.
#### Errors
| Response Code | Cause                                          |
|---------------|------------------------------------------------|
| 403 Forbidden | Missing permission `site.announcements.post`. |

--------------------------------------------------------------------------------

## {announcements_uri}
### GET
Gets a site announcement.
#### Response
An Announcement object.
#### Errors
| Response Code | Cause                                         |
|---------------|-----------------------------------------------|
| 403 Forbidden | Missing permission `site.announcements.get`. |
| 404 Not Found | No such Announcement exists.                  |

### PATCH
Updates a site announcement.
#### Request
A partial Announcement object without an ID (or author if the [users extension](./users.md) is implemented).
#### Response
The updated Announcement object.
#### Errors
| Response Code | Cause                                           |
|---------------|-------------------------------------------------|
| 403 Forbidden | Missing permission `site.announcements.patch`. |
| 404 Not Found | No such Announcement exists.                    |

### DELETE
Deletes a site announcement.
#### Errors
| Response Code | Cause                                            |
|---------------|--------------------------------------------------|
| 403 Forbidden | Missing permission `site.announcements.delete`. |
| 404 Not Found | No such Announcement exists.                     |
