Chat Moderation
===============
Implementing this extension provides users with the tools necessary to moderate the content of chat rooms.
Since this concerns chat directly, implementing this extension requires implementing the [chat extension](./chat.md).

--------------------------------------------------------------------------------

## /info
### GET
#### Response
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
| Response Code | Cause                                               |
|---------------|-----------------------------------------------------|
| 403 Forbidden | Missing permission `chat.rooms.get`.                |
| 404 Not Found | No such Chatroom exists.                            |
| 403 Forbidden | Missing permission `chat.rooms.messages.delete`.    |
| 404 Not Found | At least one ID provided has no associated message. |


## /chat/rooms/{room_id}/messages/{message_id}
### DELETE
Deletes a message.
#### Errors
| Response Code | Cause                                            |
|---------------|--------------------------------------------------|
| 403 Forbidden | Missing permission `chat.rooms.get`.             |
| 404 Not Found | No such Chatroom exists.                         |
| 403 Forbidden | Missing permission `chat.rooms.messages.delete`. |
| 404 Not Found | No such Message exists.                          |
