# [SFU (Selective Forwarding Unit)](https://bloggeek.me/webrtcglossary/sfu/)

- Media server received multiple media streams and then deciding which of these media streams should be sent to which participants.
- The number of subscribers can scale without affecting the publishers.

## Terminology

- Publisher: the one publishing or contributing a stream by sending its media towards the media server.
- Subscriber: the one receiving or viewing a stream that has been published towards the media server.

## Scenarios
- Group meetings: all peers are both, publishers of their own media streams as well as subscribers to media streams published by other participants. Each publisher sends a single (or up-to 3 video streams in case of Simulcast) and receives multiple media streams from multiple other publishers as a subscriber.
- Broadcasts: a single/few number of publishers and a larger number of subscribers.

## Characteristics
- Not all users are publishers, not all users are subscribers.
- Each user has its own set of subscribed streams.
=> Flexible, personalized due to network and device capabilities, or simply the constraint from the role of the user or their own preferences of choosing which to watch.


## Topologies

### 1. Single SFU

+ easiest to implement.
+ poor scaling to users spreading geographically.
+ too many users can increase the session sizes that exceed the capacity available of of the SFU server.
+ can act as a SPOF, a hybrid approach should be implemented: in case the Media Server is down, fallbacks to "full mesh" architecture: each publisher generates and transmits a separate video stream directly to every individual recipient in the call. But, every new participant forces the publishers to consume more CPU and uplink bandwidth to send "yet another duplicate stream". So it works for a small set of users.
=> implement Single SFU first, then scale to Cascading SFU for HA + Scaling later.

### 2. Cascading SFU:

+ multiple SFUs logically connected to serve a single session
+ users join an SFU that is closest to them
+ session sizes can exceed the capacity available in a single SFU server by having users join to different SFUs
+ can scale to 10k-1m users viewers

**MediaMTX adoptability:**
- Fully support.
- Horizontal scaling via deploying read replicas.
- Server-to-server forwarding via RTSP protocol according to official MediaMTX scalability documentation. Using WebRTC results in wasting CPU cycles, ICE negotation and heavy DTLS/SRTP encryption.

### 3. Publisher SFU:

+ also managed multiple SFUs
+ each publisher connects to the closest SFU, and subscribers connect to the relavant SFUs, regardless of geographical distance.
+ optimized for publishers, not so much for subscribers.
+ less complex to implement than Cascading SFU, but hard to monitor and control.

| Feature | Mesh | SFU | MCU |
| :--- | :--- | :--- | :--- |
| **Server cost** | Zero (no media servers) | Low to medium | High (heavy CPU use for mixing) |
| **Client CPU (sender)** | High (multiple encoders) | Low to medium (simulcast or SVC complexity) | Low (sends a single stream) |
| **Client CPU (receiver)** | High (decodes everything) | Low to medium (decodes what is selectively sent to it) | Low (decodes a single stream) |
| **Bandwidth (uplink)** | High (N-1 streams) | Low (single stream) | Low (single stream) |
| **Implementation complexity**| Low to medium | Medium to high | Low (similar to P2P) |
| **Latency** | Lowest (direct) | Low (forwarding delay) | Higher (processing delay) |
| **Scalability** | 2-5 participants | Hundreds to thousands per server | Limited by server CPU (usually low tens of participants) |
| **Layout control** | Client-side (flexible) | Client-side (flexible) | Server-side (fixed and rigid) |
