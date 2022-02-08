# Getting Started

This is internal document for clients to reference our stream service.

## Concept

In this section, we explain the room and signaling concepts. There are different types of rooms which serve different purposes. For clients, the way to interact the rooms is via signaling. Clients will need to open a websocket connection to the signaling server. Backend responsibilities and clients responsibilities will be listed. We also need all clients follow the self-check logic to detect client side problems and restart the connection if necessary. 

## Signal

In this section, we explain the signals in different signal groups. When you send a signal, this is what server expects. In general, the terminology we use in implementation is for the message from client to server or from client to client(p2p) is called as a signal. And the message from server to client is called as an ack.
