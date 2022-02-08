
# About MobZ

Low Lat HLS backend for p2m and WebRTC backend for p2p and m2m

- Infrastructure as code via serverless.framework
- Serverless signaling service
- Provide HLS/LLHLS live playback url
- WebRTC server hosted on AWS ECS
- Demo website for webrtc server testing

# Concept

Stream service uses lambdas to handle signaling. And lambdas will forward signals to webrtc sever if needed via Redis PubSub.

- p2m: live stream. one to many room.
- m2m: group chat. allow up to 9 publishers and 200 viewers in that room.
- p2p: peer to peer chat. p2p doesn't use webrtc server. lambdas forward signals between clients


## Signals

- Control Signal
  - Signals to exchange for webrtc and connection management
  - All signals are send from client to server or between clients(p2p)
  - Certain signals will recive ack from server, such as, JoinAck

```
- Webrtc Signals
  - Offer: send sdp offer
  - Answer: send sdp answer
  - iceCandidate: ice
- Room Signals
  - Join: user joins the room, first signals to be sent from the client
  - Leave: user leave the room
  - UpdateVideo: update video/audio enable/disable
  - Heartbeat: indicate the connection is alive
- M2M Only Signals
  - Kick: kick a user out of the room
  - Watch: Ask server to watch a stream and get offer sdp
  - SimulcastUpdate: viewer choose which simulcast substream to watch
  - PublishRequest: the viewer requests to become the publisher
  - PublishAccept: the room owner accept the request from viewer
  - PublishReject: the room owner reject the request from viewer
- Acks
  - JoinAck: after Join, contain ice servers, offer sdp for existed publishers, and participant tyep(you are viewer or publisher)
  - PublishListChangeAck: send to all participants when publisher changed
  - HeartbeatAck: after Heartbeat
  - WatchAck: after Watch, contains offer sdp
```

- Munit Signal
  - Signals that send from signal hanlders to webrtc servers
  - Signals handle munit life cycle

```
  - Assign: assign a station to a webrtc server
  - Unassign: unassign a station from a webrtc server
  - Drain: ban a webrtc server to accept new station, wait for existed connections to dead out
  - Revive: allow a draining webrtc server to accept new station again
  - Stop: stop the webrtc server
```

- Data Signals
  - Contain hls segment/part raw data in base64
  - Used to compose the hls playlist
  - Data is saved in Redis


## Room and Station

- Room: signaling level concept. a room can have many publishers and viewers.
- Station: webrtc server concept. a room is splitted into many station. each station can contain only one publisher. And this is the basic unit we assign to webrtc servers. Therefore, for the same room, we may have stations which run on different webrtc servers.

## Webrtc Server

Webrtc server is required for m2m. We achieve via AWS ECS. Each ECS fargate task instance represent a combination of Janus webrtc server and control unit. We call this combination as `Munit`.

## HLS

HLS playlist follow this [specification](https://tools.ietf.org/html/draft-pantos-hls-rfc8216bis-07#section-4.4.4.7)

## Access this service from outside

- Connect via websocket:
  - Signal allow to be accessed with ApiGateway websocket.

# Structure

```
root
├── docker: docker containers
├── packages: common code share across docker and lambda
├── resources: aws resources needed for this service
├── src: lambdas handling control signals
└── test: test related topic such as demo, load testing scripts and etc
```

# Dockers

```
root
└── docker
    ├── janus-media-server: Janus server
    └── regulator: Janus client which listen event from Radis and update to Janus
```

update docker to ECR and create ECS Fargate tasks, simple script from Makefile

```shell
make deploy
```

Each `ECS Fargate task` contain regulator docker and janus-media-server docker. So a control unit and Janus server, we call it Munit sometimes.
```
root
└── resources
    └── ecs-fargate.yml: define the fargate task
```
## Contact

derekxinzhewang@gmail.com