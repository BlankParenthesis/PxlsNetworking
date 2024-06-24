Board Notices
=============
Implementing this extension provides a way for board administrators to make long-term notices pertaining to a specific board.

For site-related notices, see the [site notices extension](./site_notices.md).

Notice objects represent notices and are defined by the following type:
```typescript
{
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

## Endpoints

### /info
#### GET
##### Response
```typescript
{
	"extensions": ["board_notices"];
}
```

--------------------------------------------------------------------------------

### {board_uri}/events?subscribe[]={events_list}
#### Server packets
##### BoardNoticeCreated
Sent when a board notice is created.
Set `subscribe[]=notices` to receive.
```typescript
{
	"type": "board-notice-created";
	"notice": Reference<Notice>;
}
```
##### NoticeUpdated
Sent when a board notice is updated.
Set `subscribe[]=notices` to receive.
```typescript
{
	"type": "board-notice-updated";
	"notice": Reference<Notice>;
}
```
##### NoticeRemoved
Sent whne a board notice is removed.
Set `subscribe[]=notices` to receive.
```typescript
{
	"type": "board-notice-removed";
	"notice": Reference<Notice>;
}
```
#### Errors
| Response Code | Cause                                       |
|---------------|---------------------------------------------|
| 403 Forbidden | Missing permission `boards.events.notices`. |

--------------------------------------------------------------------------------

### {board_uri}/notices
#### GET
Lists all board notices.
##### Response
A Paginated List of Notice References.
##### Errors
| Response Code | Cause                                     |
|---------------|-------------------------------------------|
| 403 Forbidden | Missing permission `boards.notices.list`. |

#### POST
Creates a board notice.
##### Request
A Notice object.
*NOTE: the author field from the [users extension](./users.md) should not be included here*
##### Response
The created Notice object.
##### Errors
| Response Code | Cause                                     |
|---------------|-------------------------------------------|
| 403 Forbidden | Missing permission `boards.notices.post`. |

--------------------------------------------------------------------------------

### {notice_uri}
#### GET
Gets a board notice.
##### Response
The Notice object.
##### Errors
| Response Code | Cause                                    |
|---------------|------------------------------------------|
| 403 Forbidden | Missing permission `boards.notices.get`. |

#### PATCH
Updates a board notice.
##### Request
A partial Notice object.
*NOTE: the author field from the [users extension](./users.md) should not be included here*
##### Response
The updated Notice object.
##### Errors
| Response Code | Cause                                      |
|---------------|--------------------------------------------|
| 403 Forbidden | Missing permission `boards.notices.patch`. |

#### DELETE
Deletes the board notice.
##### Errors
| Response Code | Cause                                       |
|---------------|---------------------------------------------|
| 403 Forbidden | Missing permission `boards.notices.delete`. |
