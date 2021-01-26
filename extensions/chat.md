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
Request that the server send events from the specified chatrooms.
```typescript
{
	"type": "subscribe";
	"rooms": Array<number | string>;
}
```
#### Unsubscribe
Request that the server no longer send events from the specified chatrooms.
```typescript
{
	"type": "unsubscribe";
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

--------------------------------------------------------------------------------

## /chat/info
### GET
General information on chatrooms.
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
Information on all chatrooms.
Returns an array of Chatroom objects.
#### Errors
| Response Code | Cause                                                       |
|---------------|-------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to list chatrooms. |

--------------------------------------------------------------------------------

## /chat/rooms/{room_id}
### GET
Returns the Chatroom object with the ID specified by `room_id`.
#### Errors
| Response Code | Cause                                                                    |
|---------------|--------------------------------------------------------------------------|
| 403 Forbidden | The client lacks the required privileges to view the chatroom requested. |
| 404 Not Found | No chatroom with the requested ID exists.                                |

--------------------------------------------------------------------------------

## /chat/rooms/{room_id}/messages
### GET
Previously sent messages.
Returns an array of Message objects.
#### Errors
| Response Code | Cause                                                            |
|---------------|------------------------------------------------------------------|
| 404 Not Found | No chatroom with the requested ID exists.                        |
| 403 Forbidden | The client lacks the required privileges to access chat history. |

### POST
Creates a new chat message and sends it to the chatroom.
#### Request
```typescript
{
	"message": string;
}
```
#### Response
The created message object.
#### Errors
| Response Code | Cause                                                      |
|---------------|------------------------------------------------------------|
| 404 Not Found | No chatroom with the requested ID exists.                  |
| 403 Forbidden | The client lacks the required privileges to post messages. |

--------------------------------------------------------------------------------

## /chat/rooms/{room_id}/messages/{message_id}
### GET
A chat message.
Returns a Message object.
#### Errors
| Response Code | Cause                                                            |
|---------------|------------------------------------------------------------------|
| 404 Not Found | No chatroom with the requested ID exists.                        |
| 403 Forbidden | The client lacks the required privileges to access chat history. |
| 404 Not Found | No message with the requested ID exists.                         |