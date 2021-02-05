Announcements
=============
Implementing this extension provides a way for server administrators to make long-term announcements which inform users of something.

Announcements are related to the current board primarily but may also be marked as global to indicate their relation more to the state of the host site or server.

Announcement objects represent announcements and are defined by the following type:
```typescript
{
	"id": number | string;
	"title": string;
	"content": string;
	"createdAt": Timestamp;
	"expiresAt": Timestamp;
	"global": boolean;
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
	"extensions": ["announcements"];
}
```

--------------------------------------------------------------------------------

## /ws?extensions[]=announcements
### Server packets
#### AnnouncementCreated
There is a new announcement.
```typescript
{
	"type": "announcement-created";
	"announcement": Announcement;
}
```
#### AnnouncementUpdated
An announcement has been updated.
```typescript
{
	"type": "announcement-updated";
	"announcement": Announcement;
}
```
#### AnnouncementRemoved
An announcement has been removed.
```typescript
{
	"type": "announcement-removed";
	"id": number | string;
}
```
### Errors
| Response Code | Cause                                      |
|---------------|--------------------------------------------|
| 403 Forbidden | Missing permission `socket.announcements`. |

--------------------------------------------------------------------------------

## /board/announcements
### GET
Lists all announcements.
#### Response
A Paginated List of Announcement objects.
#### Errors
| Response Code | Cause                                          |
|---------------|------------------------------------------------|
| 403 Forbidden | Missing permission `board.announcements.list`. |

### POST
Creates an announcement.
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
Gets an announcement.
#### Response
An Announcement object.
#### Errors
| Response Code | Cause                                         |
|---------------|-----------------------------------------------|
| 403 Forbidden | Missing permission `board.announcements.get`. |
| 404 Not Found | No such Announcement exists.                  |

### PATCH
Updates an announcement.
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
Deletes an announcement.
#### Errors
| Response Code | Cause                                            |
|---------------|--------------------------------------------------|
| 403 Forbidden | Missing permission `board.announcements.delete`. |
| 404 Not Found | No such Announcement exists.                     |
