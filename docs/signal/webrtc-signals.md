# WebRTC Signals

> **Note**: Regarding on the WebSocket Api Payload, different signal shall use different `action` and fill the signal to `data`. This is to meet the AWS ApiGateway requirement. This payload structure only apply when send the signal from the client to the signaling backend. In case of p2p signal forwarding, only the signal itself will be received by the peer, which meant you don't need to worry for decoding websocket payload structure in the client side.

### Offer Signal

```json
// WebSocket Api Payload
{
  "action": "offer",
  "data": {...}
}
```


```json
// Signal Structure(data)
{
  "s": "OFFER",
  "ch": "m2m" | "a2m" | "p2p", // room type
  // session info
  "ss": {
    "userId": String,
    "sessionId": String,
    // Session category
    "category": "stream" | "screenshare",
    // Device type
    "host": "iOS" | "iOS_EX" | "Android" | "Web" | "Web_Studio" | "Server",
    "deviceId": String
  },
  "rid": String, // room id
  // create webrtc connection to this session. In p2p, you can use just peer user id. In m2m, sessionId and userId if you start your uplink connection as a publisher
  "sdp": String, // SDP offer
  // optional info
  "op": {
    "restarting": Bool, // must by true if you restarting the uplink connection
    "hlsForwarding": Bool // enable hls forwarding for this stream
  }
}
```

```json
// Example
{
  "data": {
    "rid": "eca0609f-cf5d-42be-b4e2-dba0fa05f2e4",
    "ss": {
      "userId": "b4044fc2-1329-4de1-85e5-95f2229dcef3",
      "host": "iOS",
      "category": "stream",
      "sessionId": "22718907-D43A-46DA-95A1-DEFC73047715",
      "deviceId": "B2DC9D27-93BC-45E9-A2A9-4C3B60FCC9F6"
    },
    "ch": "m2m",
    "sdp": "v=0\r\no=- 3002825352830109796 2 IN IP4 127.0.0.1\r\ns=-\r\nt=0 0\r\na=group:BUNDLE...",
    "s": "OFFER",
    "op": {
      "restarting": false,
      "hlsForwarding": false
    }
  },
  "action": "offer"
}
```

### Offer Ack

This is used in m2m and a2m when a publisher start their own uplink connection by send an offer signal.


```json
{
  "c": 200,
  "s": "OFFER",
  "b": {
    "rid": String,
    "sdp": String // SDP answer, use this to create webrtc connection
  }
}
```

Error Ack

```json
{
  "c": 500 | 2001 | 2002 | 4001 | 4003 | 4051,
  "s": "OFFER",
  "b": {
    "req": {...}, // original signal
    "msg": String
  }
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
// WebSocket Api Payload
{
  "action": "answer",
  "data": {...}
}
```

```json
// Signal Structure(data)
{
  "s": "ANSWER",
  "ch": "m2m" | "a2m" | "p2p", // room type
  // session info
  "ss": {
    "userId": String,
    "sessionId": String,
    // Session category
    "category": "stream" | "screenshare",
    // Device type
    "host": "iOS" | "iOS_EX" | "Android" | "Web" | "Web_Studio" | "Server",
    "deviceId": String
  },
  "rid": String, // room id
  "sdp": String // SDP answer
}
```

Error Ack

```json
{
  "c": 500 | 2001 | 2002 | 4001 | 4003 | 4024,
  "s": "ANSWER",
  "b": {
    "req": {...}, // original signal
    "msg": String
  }
}
```

```
500(INTERNAL_SERVER_ERROR): Server failed, the client shall exit the room. 
2001(INVALID_SIGNAL): The signal is invalid.
2002(UNSUPPORTED_CHANNEL): The signal is not supported in this channel.
4001(ROOM_NOT_FOUND): The room doesn't exist.
4003(RECIPIENT_NOT_MEMBER): The p2p peer user is not in the room.
4024(USER_LEFT): The user has left the room.
```

### Ice Signal

```json
// WebSocket Api Payload
{
  "action": "iceCandidate",
  "data": {...}
}
```

```json
// Signal Structure(data)
{
  "s": "ICE",
  "ch": "m2m" | "a2m" | "p2p", // room type
  // session info
  "ss": {
    "userId": String,
    "sessionId": String,
    // Session category
    "category": "stream" | "screenshare",
    // Device type
    "host": "iOS" | "iOS_EX" | "Android" | "Web" | "Web_Studio" | "Server",
    "deviceId": String
  },
  "rid": String, // room id
  "sdp": String // ice candidate
}
```

Error Ack

```json
{
  "c": 500 | 2001 | 2002 | 4001 | 4003,
  "s": "ICE",
  "b": {
    "req": {...}, // original signal
    "msg": String
  }
}
```

```
500(INTERNAL_SERVER_ERROR): Server failed, the client shall exit the room. 
2001(INVALID_SIGNAL): The signal is invalid.
2002(UNSUPPORTED_CHANNEL): The signal is not supported in this channel.
4001(ROOM_NOT_FOUND): The room doesn't exist.
4003(RECIPIENT_NOT_MEMBER): The p2p peer user is not in the room.
```

### Watch Signal

Request the backend to provide offer SDP

```json
// WebSocket Api Payload
{
  "action": "watch",
  "data": {...}
}
```

```json
// Signal Structure(data)
{
  "s": "WATCH",
  "ch": "m2m" | "a2m" , // room type
  // session info
  "ss": {
    "userId": String,
    "sessionId": String,
    // Session category
    "category": "stream" | "screenshare",
    // Device type
    "host": "iOS" | "iOS_EX" | "Android" | "Web" | "Web_Studio" | "Server",
    "deviceId": String
  },
  "rid": String, // room id
  "to": {
    "userId": String, // the user id which shall be watched
    "sessionId": String, // the session id which shall be watched
  }
}
```

### Watch Ack

Contain the offer SDP for webrtc connection

```json
{
  "c": 200,
  "s": "WATCH",
  "b": {
    "pps": [
      {
        "id": String,
        "sdp": String, // SDP offer, use this to create webrtc connection
      }
    ],
    "rid": String
  }
}
```

Error Ack

```json
{
  "c": 500 | 2001 | 2002 | 4001 | 4024,
  "s": "WATCH",
  "b": {
    "req": {...}, // original signal
    "msg": String
  }
}
```

```
500(INTERNAL_SERVER_ERROR): Server failed, the client shall retry. 
2001(INVALID_SIGNAL): The signal is invalid.
2002(UNSUPPORTED_CHANNEL): The signal is not supported in this channel.
4001(ROOM_NOT_FOUND): The room doesn't exist.
4024(USER_LEFT): The user has left the room.
```