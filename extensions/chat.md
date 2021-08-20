Chat
====
Implementing this extensions adds support for real-time chat.

At the core of chat is the Message object defined as follows:
```typescript
{
	"id": number | string;
	"createdAt": Timestamp;
	"raw"?: string;
	"message": string;
}
``` 

Messages are grouped by Chatroom objects which are defined below:
```typescript
{
	"id": number | string;
	"name": string;
	"createdAt": Timestamp;
}
```

Implementations may choose to transparently censor messages.
In this case, `message` should be the censored message.
Additionally, `raw` may be sent to users with the privileges to view the original uncensored message.

If the [users extension](./users.md) is implemented, Message objects may also specify the User object of the message author:
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
	"extensions": ["chat"];
}
```

--------------------------------------------------------------------------------

## /ws?extensions[]=chat
### Client packets
#### ChatSubscribe
Request that the server send events from the specified chatrooms.
```typescript
{
	"type": "chat-subscribe";
	"rooms": Array<number | string>;
}
```
#### ChatUnsubscribe
Request that the server no longer send events from the specified chatrooms.
```typescript
{
	"type": "chat-unsubscribe";
	"rooms": Array<number | string>;
}
```

### Server packets
These packets are sent by the server to inform the client of something.
#### Messages
New messages were sent.
```typescript
{
	"type": "messages";
	"room": number | string;
	"messages": Message[];
}
```

#### Error
The client has made an error.
##### InvalidRoom
The client referenced an invalid room.
If multiple rooms were invalid, the server need only mention one.
```typescript
{
	"type": "error";
	"error": "INVALID_ROOM";
	"room": number | string;
	"cause": "FORBIDDEN" | "MISSING";
}
```
### Errors
| Response Code | Cause                                          |
|---------------|------------------------------------------------|
| 403 Forbidden | Missing permission `socket.chat`.              |

--------------------------------------------------------------------------------

## /chat/info
### GET
Gets general information on chatrooms.
#### Response
```typescript
{
	"defaultRoom"?: number | string;
	"characterLimit": number;
	"customEmoji": Array<{
		"emoji": string;
		"name": string;
	}>;
}
```

--------------------------------------------------------------------------------

## /chat/rooms
### GET
Lists all chatrooms.
#### Response
A Paginated List of Chatroom objects.
#### Errors
| Response Code | Cause                                 |
|---------------|---------------------------------------|
| 403 Forbidden | Missing permission `chat.rooms.list`. |

--------------------------------------------------------------------------------

## /chat/rooms/{room_id}
### GET
Gets a chatroom.
#### Response
A Chatroom object.
#### Errors
| Response Code | Cause                                |
|---------------|--------------------------------------|
| 404 Not Found | No such Chatroom exists.             |
| 403 Forbidden | Missing permission `chat.rooms.get`. |

--------------------------------------------------------------------------------

## /chat/rooms/{room_id}/messages
### GET
Lists messages in a chatroom.
#### Response
A Paginated List of Message objects.
#### Errors
| Response Code | Cause                                          |
|---------------|------------------------------------------------|
| 404 Not Found | No such Chatroom exists.                       |
| 403 Forbidden | Missing permission `chat.rooms.get`.           |
| 403 Forbidden | Missing permission `chat.rooms.messages.list`. |

### POST
Creates a chat message.
#### Request
```typescript
{
	"message": string;
}
```
#### Response
The created message object.
#### Errors
| Response Code | Cause                                          |
|---------------|------------------------------------------------|
| 404 Not Found | No such Chatroom exists.                       |
| 403 Forbidden | Missing permission `chat.rooms.get`.           |
| 403 Forbidden | Missing permission `chat.rooms.messages.post`. |

--------------------------------------------------------------------------------

## /chat/rooms/{room_id}/messages/{message_id}
### GET
Gets a message in a chatroom.
#### Response
A Message object.
#### Errors
| Response Code | Cause                                         |
|---------------|-----------------------------------------------|
| 404 Not Found | No such Chatroom exists.                      |
| 403 Forbidden | Missing permission `chat.rooms.get`.          |
| 404 Not Found | No such Message exists.                       |
| 403 Forbidden | Missing permission `chat.rooms.messages.get`. |