Chat
====
Implementing this extensions adds support for real-time chat.

At the core of chat is the Message object defined as follows:
```typescript
{
	"createdAt": Timestamp;
	"raw"?: string;
	"message": string;
}
``` 

Messages are grouped by Chatroom objects which are defined below:
```typescript
{
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

If the [roles extension](./roles.md) is implemented, the following permissions are added due to this extension:

| Permission                 | Purpose                                                               |
|----------------------------|-----------------------------------------------------------------------|
| `chat.rooms.list`          | Allows GET requests to `/chat/rooms`.                                 |
| `chat.rooms.get`           | Allows GET requests to `/chat/rooms/{room_id}`.                       |
| `chat.rooms.messages.list` | Allows GET requests to `/chat/rooms/{room_id}/messages`.              |
| `chat.rooms.messages.get`  | Allows GET requests to `/chat/rooms/{room_id}/messages/{message_id}`. |
| `chat.rooms.messages.post` | Allows POST requests to `/chat/rooms/{room_id}/messages`.             |

--------------------------------------------------------------------------------

## /info
### GET
```typescript
{
	"extensions": ["chat"];
}
```

--------------------------------------------------------------------------------

## /chat/ws
Websocket connection point for chat messages.
### Client packets
#### Subscribe
Request that the server send events from the chatrooms specified by URI.
```typescript
{
	"type": "subscribe";
	"rooms": string[];
}
```
#### Unsubscribe
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

--------------------------------------------------------------------------------

## /chat/info
### GET
General information on chatrooms.
```typescript
{
	"defaultRoom"?: Reference<Chatroom>;
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
A list of all chatrooms.
#### Request
An array of Chatroom References.
#### Errors
| Response Code | Cause                                                       |
|---------------|-------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to list chatrooms. |

--------------------------------------------------------------------------------

## {chatroom_uri}
### GET
#### Request
The Chatroom object.
#### Errors
| Response Code | Cause                                                                    |
|---------------|--------------------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to view the chatroom requested. |

--------------------------------------------------------------------------------

## {chatroom_uri}/messages
### GET
A list of all messages in the chatroom.
#### Response
An array of Message References.
#### Errors
| Response Code | Cause                                                            |
|---------------|------------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to access chat history. |

### POST
Create a new chat message and sends it to the chatroom.
#### Request
```typescript
{
	"message": string;
}
```
#### Response
A Reference to the created message object.
#### Errors
| Response Code | Cause                                                      |
|---------------|------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to post messages. |

--------------------------------------------------------------------------------

## {message_uri}
### GET
#### Request
The Message object.
#### Errors
| Response Code | Cause                                                            |
|---------------|------------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to access chat history. |