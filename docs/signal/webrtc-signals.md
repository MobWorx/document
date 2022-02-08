# WebRTC Signals

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
    "s": "OFFER_ACK",
    "d": {
      "rid": String,
      "sdp": String // SDP answer, use this to create webrtc connection
    }
  },
  "ch": "m2m" | "a2m", // room type
}
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
    "s": "WATCH_ACK",
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