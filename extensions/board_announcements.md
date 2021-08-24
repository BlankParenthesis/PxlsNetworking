Board Announcements
===================
Implementing this extension provides a way for board administrators to make long-term announcements which inform users of something.

For site-related announcements, see the [site announcements extension](./site_announcements.md).

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
	"extensions": ["board_announcements"];
}
```

--------------------------------------------------------------------------------

## /ws?extensions[]=board_announcements
### Server packets
#### BoardAnnouncementCreated
There is a new board announcement.
```typescript
{
	"type": "board-announcement-created";
	"announcement": Announcement;
}
```
#### AnnouncementUpdated
A board announcement has been updated.
```typescript
{
	"type": "board-announcement-updated";
	"announcement": Announcement;
}
```
#### AnnouncementRemoved
A board announcement has been removed.
```typescript
{
	"type": "board-announcement-removed";
	"id": number | string;
}
```
### Errors
| Response Code | Cause                                      |
|---------------|--------------------------------------------|
| 403 Forbidden | Missing permission `socket.board.announcements`. |

--------------------------------------------------------------------------------

## /board/announcements
### GET
Lists all board announcements.
#### Response
A Paginated List of Announcement objects.
#### Errors
| Response Code | Cause                                          |
|---------------|------------------------------------------------|
| 403 Forbidden | Missing permission `board.announcements.list`. |

### POST
Creates a board announcement.
#### Request
An Announcement object without an ID (or author if the [users extension](./users.md) is implemented).
#### Response
The created Announcement object.
#### Errors
| Response Code | Cause                                          |
|---------------|------------------------------------------------|
| 403 Forbidden | Missing permission `board.announcements.post`. |

--------------------------------------------------------------------------------

## /board/announcements/{announcements_id}
### GET
Gets a board announcement.
#### Response
An Announcement object.
#### Errors
| Response Code | Cause                                         |
|---------------|-----------------------------------------------|
| 403 Forbidden | Missing permission `board.announcements.get`. |
| 404 Not Found | No such Announcement exists.                  |

### PATCH
Updates a board announcement.
#### Request
A partial Announcement object without an ID (or author if the [users extension](./users.md) is implemented).
#### Response
The updated Announcement object.
#### Errors
| Response Code | Cause                                           |
|---------------|-------------------------------------------------|
| 403 Forbidden | Missing permission `board.announcements.patch`. |
| 404 Not Found | No such Announcement exists.                    |

### DELETE
Deletes a board announcement.
#### Errors
| Response Code | Cause                                            |
|---------------|--------------------------------------------------|
| 403 Forbidden | Missing permission `board.announcements.delete`. |
| 404 Not Found | No such Announcement exists.                     |
