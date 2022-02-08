# Room Signals

Common signals for room control.

## Room Operation Signals

These signals are used for all rooms.

### Join Signal

This is the first signal that you need to send.

```json
{
  "action": "join",
  "data": {
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
}
```

### Join Ack

The server will return room info and publisher list.

```json
{
  "c": 200,
  "b": {
    "s": "JOIN_ACK",
    "d": {
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
  },
  "ch": "m2m" | "p2p" | "a2m" | "p2m", // room type
}
```

Error Ack

```json
{
  "c": 500 | 4001 | 4002 | 4003 | 4004 | 4005,
  "b": {
    "s": "JOIN_ACK",
    "d": {
      "uid": String,
      "rid": String,
      "msg": String
    }
  },
  "ch": "m2m" | "p2p" | "a2m" | "p2m", // room type
}
```

```
500 : internalServerError
4001 : roomNotFound
4002 : notRoomMember
4003 : recipientNotMember
4004 : alreadyInRoom
4005 : actionNotSupported
```

### Leave Signal

Send this signal when leave the room so that backend can clean up the resource.

```json
{
  "action": "leave",
  "data": {
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
}
```

### Update Video Signal

When the publisher change video and audio status, send this signal to notify the backend.

```json
{
  "action": "updateVideo",
  "data": {
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
}
```

### Publish List Changed Ack

Whenever publisher list change, inlcude video/audio status change, the server will send this signal to notify the client.

```json
{
  "c": 200,
  "b": {
    "s": "PUBLISH_LIST_CHANGED_ACK",
    "d": {
      // latest publisher list
      "publisher": [
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
  },
  "ch": "m2m" | "p2p" | "a2m" | "p2m", // room type
}
```

### Heartbeat Signal

Send this signal to keep the client presence in current room.

```json
{
  "action": "heartbeat",
  "data": {
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
}
```

### Heartbeat Ack

The server will send back latest publisher list along with the ack.

```json
{
  "c": 200,
  "b": {
    "s": "HEARTBEAT_ACK",
    "d": {
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
  },
  "ch": "m2m" | "p2p" | "a2m" | "p2m", // room type
}
```

## P2M, M2M and A2M Operation Signals

### Kick Signal

Remove a user from the room.

```json
{
  "action": "kick",
  "data": {
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
}
```

## M2M and A2M Operation Signals

### Publish Request Signal

The viewer can request to promote their status to publisher. The signal will be forwarded to the current room owner.

```json
{
  "action": "publishRequest",
  "data": {
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
}
```

### Publish Accept Signal

The owner accept the publish request. The backend will promote the viewer status to publisher and forward this signal to the new publisher. Upon recive this signal, the new publisher can start to publish stream.

```json
{
  "action": "publishAccept",
  "data": {
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
}
```

### Publish Reject Signal

The owner reject the publish request. he backend will forward this signal to the viewer.

```json
{
  "action": "publishReject",
  "data": {
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
}
```
