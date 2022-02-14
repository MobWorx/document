# WebRTC Signals

> **Note**: Regarding on the Ack, in the previous version, we use signal name plus `ACK`, such as `JOIN_ACK`. But in the future version, we will just use signal name. So the ack for join signal will be same as signal name `JOIN`. All ack will contain code which can help clients to distinguish it from signals. For now, the client shall consider for both cases, therefore, `JOIN` and `JOIN_ACK` are both valid ack. we will deprecate `XXX_ACK` in the future.

### Offer Signal

```json
{
  "action": "offer",
  "data": {
    "s": "OFFER",
    "ch": "m2m" | "a2m" | "p2p", // room type
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
    // create webrtc connection to this user. In p2p, this will be your peer user id. In m2m, this is self userId if you start your uplink connection as a publisher
    "to": String, 
    "sdp": String, // SDP offer
    // optional info
    "op": {
      "restarting": Bool, // must by true if you restarting the uplink connection
      "hlsForwarding": Bool // enable hls forwarding for this stream
    }
  }
}
```

### Offer Ack

This is used in m2m and a2m when a publisher start their own uplink connection by send an offer signal.


```json
{
  "c": 200,
  "b": {
    "s": "OFFER_ACK", // will be "OFFER" in next version
    "d": {
      "rid": String,
      "sdp": String // SDP answer, use this to create webrtc connection
    }
  },
  "ch": "m2m" | "a2m", // room type
}
```

Error Ack

```json
{
  "c": 500 | 2001 | 2002 | 4001 | 4003 | 4051,
  "b": {
    "s": "OFFER_ACK", // will be "OFFER" in next version
    "d": {
      "rid": String,
      "msg": String
    }
  },
  "ch": "m2m" | "p2p" | "a2m", // room type
}
```

```
500(INTERNAL_SERVER_ERROR): Server failed, the client shall exit the room. 
2001(INVALID_SIGNAL): The signal is invalid.
2002(UNSUPPORTED_CHANNEL): The signal is not supported in this channel.
4001(ROOM_NOT_FOUND): The room doesn't exist.
4003(RECIPIENT_NOT_MEMBER): The p2p peer user is not in the room.
4051(PERMISSION_NOT_FOUND): The user doesn't have the permission, only publisher can send this signal.
```

### Answer Signal

```json
{
  "action": "answer",
  "data": {
    "s": "ANSWER",
    "ch": "m2m" | "a2m" | "p2p", // room type
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
    "to": String, // create webrtc connection to this user id
    "sdp": String, // SDP answer
  }
}
```

Error Ack

```json
{
  "c": 500 | 2001 | 2002 | 4001 | 4003 | 4051,
  "b": {
    "s": "ANSWER",
    "d": {
      "rid": String,
      "msg": String
    }
  },
  "ch": "m2m" | "p2p" | "a2m", // room type
}
```

```
500(INTERNAL_SERVER_ERROR): Server failed, the client shall exit the room. 
2001(INVALID_SIGNAL): The signal is invalid.
2002(UNSUPPORTED_CHANNEL): The signal is not supported in this channel.
4001(ROOM_NOT_FOUND): The room doesn't exist.
4003(RECIPIENT_NOT_MEMBER): The p2p peer user is not in the room.
4051(PERMISSION_NOT_FOUND): The user doesn't have the permission, only publisher can send this signal.
```

### Ice Signal

```json
{
  "action": "iceCandidate",
  "data": {
    "s": "ICE",
    "ch": "m2m" | "a2m" | "p2p", // room type
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
    "to": String, // webrtc connection with this user id
    "sdp": String, // ice candidate
  }
}
```

Error Ack

```json
{
  "c": 500 | 2001 | 2002 | 4001 | 4003 | 4051,
  "b": {
    "s": "ICE",
    "d": {
      "rid": String,
      "msg": String
    }
  },
  "ch": "m2m" | "p2p" | "a2m", // room type
}
```

```
500(INTERNAL_SERVER_ERROR): Server failed, the client shall exit the room. 
2001(INVALID_SIGNAL): The signal is invalid.
2002(UNSUPPORTED_CHANNEL): The signal is not supported in this channel.
4001(ROOM_NOT_FOUND): The room doesn't exist.
4003(RECIPIENT_NOT_MEMBER): The p2p peer user is not in the room.
4051(PERMISSION_NOT_FOUND): The user doesn't have the permission, only publisher can send this signal.
```

### Watch Signal

Request the backend to provide offer SDP

```json
{
  "action": "watch",
  "data": {
    "s": "WATCH",
    "ch": "m2m" | "a2m" , // room type
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
    "to": String // the user to create connectino with
  }
}
```

### Watch Ack

Contain the offer SDP for webrtc connection

```json
{
  "c": 200,
  "b": {
    "s": "WATCH_ACK", // will be "WATCH" in next version
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
      "rid": String
    }
  },
  "ch": "m2m" | "a2m", // room type
}
```

Error Ack

```json
{
  "c": 500 | 2001 | 2002 | 4001,
  "b": {
    "s": "ANSWER",
    "d": {
      "rid": String,
      "msg": String
    }
  },
  "ch": "m2m" | "p2p" | "a2m", // room type
}
```

```
500(INTERNAL_SERVER_ERROR): Server failed, the client shall retry. 
2001(INVALID_SIGNAL): The signal is invalid.
2002(UNSUPPORTED_CHANNEL): The signal is not supported in this channel.
4001(ROOM_NOT_FOUND): The room doesn't exist.
```