Chat
====
Implementing this extensions adds support for real-time chat.

At the core of chat is the Message object defined as follows:
```typescript
{
	"created_at": Timestamp;
	"raw"?: string;
	"message": string;
}
``` 

Messages are grouped by Chatroom objects which are defined below:
```typescript
{
	"name": string;
	"created_at": Timestamp;
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
Request that the server send events from the chatrooms specified by URI.
```typescript
{
	"type": "subscribe";
	"rooms": string[];
}
```
#### ChatUnsubscribe
Request that the server no longer send events from the chatrooms specified by URI.
```typescript
{
	"type": "unsubscribe";
	"rooms": string[];
}
```

### Server packets
These packets are sent by the server to inform the client of something.
#### Messages
New messages were sent.
```typescript
{
	"type": "messages";
	"room": Reference<Chatroom>;
	"messages": Reference<Message>[];
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
	"room": string;
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
	"character_limit": number;
	"custom_emoji": Array<{
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
An array of Chatroom References.
#### Errors
| Response Code | Cause                                 |
|---------------|---------------------------------------|
| 403 Forbidden | Missing permission `chat.rooms.list`. |

--------------------------------------------------------------------------------

## /chat/rooms/default/
Requests made to this endpoint should be redirected using HTTP status 307 to the room object of the default chat.
If any sub-endpoints are called on this, the should likewise be redirected, keeping the path.

--------------------------------------------------------------------------------

## {chatroom_uri}
### GET
Gets a chatroom.
#### Response
The Chatroom object.
#### Errors
| Response Code | Cause                                |
|---------------|--------------------------------------|
| 403 Forbidden | Missing permission `chat.rooms.get`. |

--------------------------------------------------------------------------------

## {chatroom_uri}/messages
### GET
Lists messages in a chatroom.
#### Response
A Paginated List of Message References.
#### Errors
| Response Code | Cause                                          |
|---------------|------------------------------------------------|
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
A Reference to the created message object.
#### Errors
| Response Code | Cause                                          |
|---------------|------------------------------------------------|
| 403 Forbidden | Missing permission `chat.rooms.get`.           |
| 403 Forbidden | Missing permission `chat.rooms.messages.post`. |

--------------------------------------------------------------------------------

## {message_uri}
### GET
Gets a message in a chatroom.
#### Response
The Message object.
#### Errors
| Response Code | Cause                                         |
|---------------|-----------------------------------------------|
| 403 Forbidden | Missing permission `chat.rooms.get`.          |
| 403 Forbidden | Missing permission `chat.rooms.messages.get`. |
