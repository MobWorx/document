# WebRTC

We use webrtc to provide services for the use cases which require high latency requirement. Including video call and group chat.

Currently webrtc supports:
- Group Chat Room(m2m)
- Audio Only Group Chat Room(a2m)
- Peer to Peer Chat Room(p2p)

Among the above three rooms, we provide SFU webrtc server for Group Chat Room(m2m) and Audio Only Group Chat Room(a2m). Peer to Peer Chat Room(p2p) is a direct webrtc connetion between two peers.

## SDK

We are using customized Google WebRTC SDK for iOS. For Android, we will package our customized WebRTC SDK build. For web, we recommand to use the browser build-in webrtc sdk.

Our customized WebRTC SDK provides the following features:
1. Customized audio source.
2. Extra statistics for stream tracking
3. Audio level detection
4. Customized simulcast logic

It contains no breaking changes from the official webrtc sdk.

## Signaling

Our room signaling server also serves as the webrtc signaling server.
