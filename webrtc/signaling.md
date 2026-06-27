# Signaling

Coordinate communication between 2+ peers before a real-time media session begins. 

Three types of messages:

- Session descriptions via SDP offer/answer.
- ICE candidates for NAT traversal
- Application-level metadata: custom events like join, leave, mute, screen-share
## Procedures

1. The initiaing peer creates an SDP offer using `RTCPeerConnection.createOffer()` API, set it as its local description.
2. This SDP offer is sent to the remote peer through a *signaling channel*.
3. The remote peer receives the SDP offer, sets it as its remote description, generates an SDP answer with `RTCPeerConnection.createAnswer()` API.
4. This SDP answer is sent to the initiating peer through the same signaling channel.
5. Simultaneously, both peers gather ICE candidates and exchange them through the same signaling channel.
6. Once both sides have a complete set of session descriptions and ICE candidates, a direct P2P media connection is established

The signaling channel requires a signaling server. This server does not handle any media traffic and acts purely as a relay for metadata exchange.

Once the direct P2P media connection is up, the signaling channel can stay opened for renegotiation, or closed.

## Signaling Transport Protocols

3 protocols:

1. XHR: 

- Client-to-server over HTTP 
- HTTP polling/long-polling.
- Simple to implement, adds latency due to req-res cycle.
- Used by WHIP/WHEP.

2. SSE: 

- Server-to-client over HTTP
- Lower latency than HTTP polling, simpler than WebSocket.

3. WebSocket:

- LL, full-duplex/bi-directional, persistent connection, efficient message framinig.
- Production-ready.
- Hybrid: WebSocket for primary signalling flow, XHR as fallbacks where WebSocket is blocked.

## Signaling Data Protocols

1. Proprietary/Custom JSON: define your own message types (offer, answer, ice-candidate, join, leave, ...) and handle them in application logic. Transfer over WebSockets.
2. SIP (Session Initiation Protocol) over WebSockets.
3. XMPP (Extensible Messaging and Presence Protocol)/Jingle: suited for apps that already used XMPP for messaging and presence.
4. Matrix: open, federated communication protocol. 
