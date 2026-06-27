# Session

## MediaMTX behavior

1. Standard Unicast (WebRTC, RTMP, Unicast RTSP)

1 session (+1 encryption handshake) from publisher to server
N sessions (+N DTLS/SRTP handshake) from server to subscribers

Cons:
- Server's CPU packetize, track network state and encrypt the media for every single subscriber 
=> CPU, RAM and Uplink bandwidth utilization scales linearly with the subscribers count

Pros:
- Remain standard for public-facing deployments.


2. RTSP with UDP Multicast

1 session from publisher to server
1 broadcasted session from server to N subscribers.

Pros:
- Server prepares and transmits the media packet exactly once => Optimized CPU+RAM.
- Uplink bandwidth for one stream is required. => Optimized bandwidth as well.

**The catch:**
- Routers and Switches must be configured with IGMP snooping. They now hold the responsiblity to duplicate the packets and route them to subscribers.
- Confine to local networks only. (not a problem on my end)

