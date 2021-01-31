Chat Moderation
===============
Implementing this extension provides users with the tools necessary to moderate the content of chat rooms.
Since this concerns chat directly, implementing this extension requires implementing the [chat extension](./chat.md).


If the [roles extension](./roles.md) is implemented, the following permissions are added due to this extension:

| Permission                   | Purpose                                                                                                       |
|------------------------------|---------------------------------------------------------------------------------------------------------------|
| `chat.rooms.messages.delete` | Allows DELETE requests to `/chat/rooms/{room_id}/messages` and `/chat/rooms/{room_id}/messages/{message_id}`. |

--------------------------------------------------------------------------------

## /info
### GET
```typescript
{
	"extensions": ["chat_moderation"];
}
```

--------------------------------------------------------------------------------

## /ws?extensions[]=chat_moderation
### Server packets
#### Delete
Messages were deleted by a moderator.
```typescript
{
	"type": "delete";
	"room": number | string;
	"messages": Array<number | string>;
}
```
--------------------------------------------------------------------------------


## /chat/rooms/{room_id}/messages
### DELETE
Deletes all specified messages.
#### Request
```typescript
{
	messages: Array<number | string>;
}
```
#### Errors
| Response Code | Cause                                                                |
|---------------|----------------------------------------------------------------------|
| 403 Forbidden | The client does not have the required privileges to delete messages. |
| 404 Not Found | No chat room with the specified ID exists.                           |
| 404 Not Found | At least one of the IDs provided has no associated message.          |


## /chat/rooms/{room_id}/messages/{message_id}
### DELETE
Deletes the specified message.
#### Errors
| Response Code | Cause                                                                |
|---------------|----------------------------------------------------------------------|
| 403 Forbidden | The client does not have the required privileges to delete messages. |
| 404 Not Found | No chat room with the specified ID exists.                           |
| 404 Not Found | No messages with the specified ID exists.                            |
