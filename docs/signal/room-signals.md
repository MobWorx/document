# Room Signals

Common signals for room control.

> **Note**: Regarding on the WebSocket Api Payload, different signal shall use different `action` and fill the signal to `data`. This is to meet the AWS ApiGateway requirement. This payload structure only apply when send the signal from the client to the signaling backend. In case of p2p signal forwarding, only the signal itself will be received by the peer, which meant you don't need to worry for decoding websocket payload structure in the client side.

## Room Operation Signals

These signals are used for all rooms.

### Join Signal

This is the first signal that you need to send.

```json
// WebSocket Api Payload
{
  "action": "join",
  "data": {...}
}
```


```json
// Signal Structure(data)
{
  "s": "JOIN",
  "ch": "m2m" | "p2p" | "a2m" | "p2m", // room type
  // session info
  "ss": {
    "userId": String,
    "sessionId": String,
    // Session category
    "category": "stream" | "screenshare",
    // Device type
    "host": "iOS" | "iOS_EX" | "Android" | "Web" | "Web_Studio",
    "deviceId": String
  },
  "uid": String, // deprecated, in favor of "ss"
  "rid": String, // room id
  "ve": Bool, // video enabled
  "ae": Bool, // audio enabled
  "eg": "native" | "agora", // which stream engine client want to use
  // optional info
  "op": {
    "roomCreating": Bool, // must by true if you start the room
    "userFaking": Bool // convenient for testing which allow userId to be random
  }
}
```

### Join Ack

The server will return room info and publisher list.

```json
{
  "c": 200,
  "s": "JOIN",
  "b": {
    "pps": [
      {
        "id": String,
        "sdp": String, // SDP offer, use this to create webrtc connection
        "video": Bool,
        "audio": Bool,
        "username": String,
        "fullname": String,
        "avatar": String, // avatar url
        "sessionId": String,
        "host": "iOS" | "iOS_EX" | "Android" | "Web" | "Web_Studio",
        "category": "stream" | "screenshare",
        "deviceId": String
      }
    ],
    "rid": String,
    // This is your role determind by the server. you can start stream if you are the publisher.
    "pt": "publisher" | "viewer",
    // Which stream engine client want to use, determind by the server.
    "eg": "native" | "agora",
    // For webrtc, this is the stun server url you shall use
    "icfg": {
      "is": [
        "stun:stun.l.google.com:19302",
        "stun:stun2.l.google.com:19302"
      ]
    }
  }
}
```

Error Ack

```json
{
  "c": 500 | 2001 | 2002 | 4001 | 4022,
  "s": "JOIN",
  "b": {
    "rid": String,
    "msg": String
  }
}
```

```
500(INTERNAL_SERVER_ERROR): Server failed, the client shall exit the room. 
2001(INVALID_SIGNAL): The signal is invalid.
2002(UNSUPPORTED_CHANNEL): The signal is not supported in this channel.
4001(ROOM_NOT_FOUND): The room doesn't exist.
4022(ALREADY_EXIST_ROOM): The room already exists. Cannot create room.
```

### Leave Signal

Send this signal when leave the room so that backend can clean up the resource.

```json
// WebSocket Api Payload
{
  "action": "leave",
  "data": {...}
}
```

```json
// Signal Structure(data)
{
  "s": "LEAVE",
  "ch": "m2m" | "p2p" | "a2m" | "p2m", // room type
  // session info
  "ss": {
    "userId": String,
    "sessionId": String,
    // Session category
    "category": "stream" | "screenshare",
    // Device type
    "host": "iOS" | "iOS_EX" | "Android" | "Web" | "Web_Studio",
    "deviceId": String
  },
  "uid": String, // deprecated, in favor of "ss"
  "rid": String, // room id
  "to": String // in p2p, this is the peer id
}
```

Error Ack

```json
{
  "c": 500 | 2001 | 2002 | 4001,
  "s": "leave",
  "b": {
   "rid": String,
    "msg": String
  }
}
```

```
500(INTERNAL_SERVER_ERROR): Server failed, the client shall exit the room. 
2001(INVALID_SIGNAL): The signal is invalid.
2002(UNSUPPORTED_CHANNEL): The signal is not supported in this channel.
4001(ROOM_NOT_FOUND): The room doesn't exist.
```

### Update Video Signal

When the publisher change video and audio status, send this signal to notify the backend.

```json
// WebSocket Api Payload
{
  "action": "updateVideo",
  "data": {...}
}
```

```json
// Signal Structure(data)
{
  "s": "UPDATE",
  "ch": "m2m" | "p2p" | "a2m" | "p2m", // room type
  // session info
  "ss": {
    "userId": String,
    "sessionId": String,
    // Session category
    "category": "stream" | "screenshare",
    // Device type
    "host": "iOS" | "iOS_EX" | "Android" | "Web" | "Web_Studio",
    "deviceId": String
  },
  "uid": String, // deprecated, in favor of "ss"
  "rid": String, // room id
  "ve": Bool, // video enabled
  "ae": Bool // audio enabled
}
```

Error Ack

```json
{
  "c": 500 | 2001 | 2002 | 4001 | 4051,
  "s": "UPDATE",
  "b": {
    "rid": String,
    "msg": String
  }
}
```

```
500(INTERNAL_SERVER_ERROR): Server failed, the client shall exit the room. 
2001(INVALID_SIGNAL): The signal is invalid.
2002(UNSUPPORTED_CHANNEL): The signal is not supported in this channel.
4001(ROOM_NOT_FOUND): The room doesn't exist.
4051(PERMISSION_NOT_FOUND): The user doesn't have the permission, only publisher can send this signal.
```

### Publish List Changed Ack

Whenever publisher list change, inlcude video/audio status change, the server will send this signal to notify the client.

```json
{
  "c": 200,
  "s": "PUBLISH_LIST_CHANGED",
  "b": {
    // latest publisher list
    "pps": [
      {
        "id": String,
        "video": Bool,
        "audio": Bool,
        "username": String,
        "fullname": String,
        "avatar": String, // avatar url
        "sessionId": String,
        "host": "iOS" | "iOS_EX" | "Android" | "Web" | "Web_Studio",
        "category": "stream" | "screenshare",
        "deviceId": String
      }
    ],
    "rid": String
  }
}
```

### Heartbeat Signal

Send this signal to keep the client presence in current room.

```json
// WebSocket Api Payload
{
  "action": "heartbeat",
  "data": {...}
}
```

```json
// Signal Structure(data)
{
  "s": "HEARTBEAT",
  "ch": "m2m" | "p2p" | "a2m" | "p2m", // room type
  // session info
  "ss": {
    "userId": String,
    "sessionId": String,
    // Session category
    "category": "stream" | "screenshare",
    // Device type
    "host": "iOS" | "iOS_EX" | "Android" | "Web" | "Web_Studio",
    "deviceId": String
  },
  "uid": String, // deprecated, in favor of "ss"
  "rid": String, // room id
  "bid": String, // heartbeat id
  "send_ack": Bool // flag, whether server shall send back ack
}
```

### Heartbeat Ack

The server will send back latest publisher list along with the ack.

```json
{
  "c": 200,
  "s": "HEARTBEAT",
  "b": {
    "pps": [
      {
        "id": String,
        "video": Bool,
        "audio": Bool,
        "username": String,
        "fullname": String,
        "avatar": String, // avatar url
        "sessionId": String,
        "host": "iOS" | "iOS_EX" | "Android" | "Web" | "Web_Studio",
        "category": "stream" | "screenshare",
        "deviceId": String
      }
    ],
    "rid": String, // room id
    "bid": String // heartbeat id
  }
}
```

Error Ack

```json
{
  "c": 500 | 2001 | 4001,
  "s": "HEARTBEAT",
  "b": {
    "rid": String,
    "msg": String
  }
}
```

```
500(INTERNAL_SERVER_ERROR): Server failed, the client shall exit the room. 
2001(INVALID_SIGNAL): The signal is invalid.
4001(ROOM_NOT_FOUND): The room doesn't exist.
```

## P2M, M2M and A2M Operation Signals

### Kick Signal

Remove a user from the room.

```json
// WebSocket Api Payload
{
  "action": "kick",
  "data": {...}
}
```

```json
// Signal Structure(data)
{
  "s": "KICK",
  "ch": "m2m" | "a2m" | "p2m", // room type
  // session info
  "ss": {
    "userId": String,
    "sessionId": String,
    // Session category
    "category": "stream" | "screenshare",
    // Device type
    "host": "iOS" | "iOS_EX" | "Android" | "Web" | "Web_Studio",
    "deviceId": String
  },
  "uid": String, // deprecated, in favor of "ss"
  "rid": String, // room id
  "to": String // the user id which shall be kicked
}
```

Error Ack

```json
{
  "c": 500 | 2001 | 2002 | 4001 | 4051,
  "s": "KICK",
  "b": {
    "rid": String,
    "msg": String
  }
}
```

```
500(INTERNAL_SERVER_ERROR): Server failed, the client shall exit the room. 
2001(INVALID_SIGNAL): The signal is invalid.
2002(UNSUPPORTED_CHANNEL): The signal is not supported in this channel.
4001(ROOM_NOT_FOUND): The room doesn't exist.
4051(PERMISSION_NOT_FOUND): The user doesn't have the permission, only publisher can send this signal.
```

## M2M and A2M Operation Signals

### Publish Request Signal

The viewer can request to promote their status to publisher. The signal will be forwarded to the current room owner.

```json
// WebSocket Api Payload
{
  "action": "publishRequest",
  "data": {...}
}
```

```json
// Signal Structure(data)
{
  "action": "publishRequest",
  "s": "PUBLISH_REQUEST",
  "ch": "m2m" | "a2m", // room type
  // session info
  "ss": {
    "userId": String,
    "sessionId": String,
    // Session category
    "category": "stream" | "screenshare",
    // Device type
    "host": "iOS" | "iOS_EX" | "Android" | "Web" | "Web_Studio",
    "deviceId": String
  },
  "uid": String, // deprecated, in favor of "ss"
  "rid": String, // room id
  "ve": Bool, // video enabled
  "ae": Bool, // audio enabled
  "msg": String // deprecated, message sent to the owner
}
```

Error Ack

```json
{
  "c": 500 | 2001 | 2002 | 4001 | 4023,
  "s": "PUBLISH_REQUEST",
  "b": {
    "rid": String,
    "msg": String
  }
}
```

```
500(INTERNAL_SERVER_ERROR): Server failed, the client shall exit the room. 
2001(INVALID_SIGNAL): The signal is invalid.
2002(UNSUPPORTED_CHANNEL): The signal is not supported in this channel.
4001(ROOM_NOT_FOUND): The room doesn't exist.
4023(ALREADY_IS_PUBLISHER): The user is a publisher already.
```

### Publish Accept Signal

The owner accept the publish request. The backend will promote the viewer status to publisher and forward this signal to the new publisher. Upon recive this signal, the new publisher can start to publish stream.

```json
// WebSocket Api Payload
{
  "action": "publishAccept",
  "data": {...}
}
```

```json
// Signal Structure(data)
{
  "s": "PUBLISH_ACCEPT",
  "ch": "m2m" | "a2m", // room type
  // session info
  "ss": {
    "userId": String,
    "sessionId": String,
    // Session category
    "category": "stream" | "screenshare",
    // Device type
    "host": "iOS" | "iOS_EX" | "Android" | "Web" | "Web_Studio",
    "deviceId": String
  },
  "uid": String, // deprecated, in favor of "ss"
  "rid": String, // room id
  "to": String // the user who request to be a publisher
}
```

Error Ack

```json
{
  "c": 500 | 2001 | 2002 | 4001 | 4002 | 4051,
  "s": "PUBLISH_ACCEPT",
  "b": {
    "rid": String,
    "msg": String
  }
}
```

```
500(INTERNAL_SERVER_ERROR): Server failed, the client shall exit the room. 
2001(INVALID_SIGNAL): The signal is invalid.
2002(UNSUPPORTED_CHANNEL): The signal is not supported in this channel.
4001(ROOM_NOT_FOUND): The room doesn't exist.
4002(NOT_ROOM_MEMBER): The promoted user is not in the room.
4051(PERMISSION_NOT_FOUND): The user doesn't have the permission, only the owner can send this signal.
```

### Publish Reject Signal

The owner reject the publish request. he backend will forward this signal to the viewer.

```json
// WebSocket Api Payload
{
  "action": "publishReject",
  "data": {...}
}
```

```json
// Signal Structure(data)
{
  "s": "PUBLISH_REJECT",
  "ch": "m2m" | "a2m", // room type
  // session info
  "ss": {
    "userId": String,
    "sessionId": String,
    // Session category
    "category": "stream" | "screenshare",
    // Device type
    "host": "iOS" | "iOS_EX" | "Android" | "Web" | "Web_Studio",
    "deviceId": String
  },
  "uid": String, // deprecated, in favor of "ss"
  "rid": String, // room id
  "to": String // the user who request to be a publisher
}
```

Error Ack

```json
{
  "c": 500 | 2001 | 2002 | 4001 | 4002,
  "s": "PUBLISH_REJECT", 
  "b": {
    "rid": String,
    "msg": String
  }
}
```

```
500(INTERNAL_SERVER_ERROR): Server failed, the client shall exit the room. 
2001(INVALID_SIGNAL): The signal is invalid.
2002(UNSUPPORTED_CHANNEL): The signal is not supported in this channel.
4001(ROOM_NOT_FOUND): The room doesn't exist.
4002(NOT_ROOM_MEMBER): The promoted user is not in the room.
```
