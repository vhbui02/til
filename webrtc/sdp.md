# SDP (Session Description Protocol)

- Data format for describing media information of a session for numerous protocols: VoIP, WebRTC (standardized by IETF and W3C), ...
- Used during Signalling phase. WHIP/WHEP extensions utilizing SDP become a well-known WebRTC signaling protocol for live streaming use case.

## Anatomy

1. WebRTC SDP (Cre: https://webrtchacks.com/sdp-anatomy/)

```
# ================================================ #
# Global Lines                                     #
# ================================================ #

# '4611731400430051336': session ID
# '2': session version, increase by one if a new offer/answer negotiation happened
# 'IN': Internet, network type
# 'IP4': IP address type
# '127.0.0.1': unicast address of the machine which created the SDP
o=- 4611731400430051336 2 IN IP4 127.0.0.1

# texture session name, not commonly used
s=-

# the timing of the session
t=0 0

# bundle groupings
# establish the relationship between several media lines included in the SDP, i.e., audio and video
# In WebRTC specifically, it's used to multiplex several media flows in the same RTP session.
# '0 1': the browser offers to multiplex the "mids" 0 and 1 (more about "mid" below). CAUTION: the otherside must also support it.
a=group:BUNDLE 0 1

# critical for WebRTC multiplexing but MediaMTX's reader.js leaves the browser's default BUNDLE setup untouched
a=msid-semantic: WMS lgsCFqt9kN2fVKw5wg3NKqGdATQoltEwOdMS

# ================================================ #
# Audio Lines                                      #
# ================================================ #

## 'm': media line
## 'audio': IANA media type
## '58779': port placeholder. Ignored in WebRTC. In classic protocol, other peers send audio data to this port. WebRTC use ICE candidates instead.
## 'UDP/TLS/RTP/SAVPF': send data over UDP, encrypt via DTLS, package media blocks using RTP, use Secure Audio-Video Profile with Feedback to handle connection reports and congestion control (RFC5764). Requires: SRTP + SRTCP + RTCP Feedback packets.
## '111 103 104 9 0 8 106 105 13 126': media format descriptions for sending/receiving media (a.k.a "payload types/payload numbers).
## - Static Payload Types (<96): mapped to IANA encoding formats. Historically, early VoIP standards assigned fixed numbers to old, common codecs so browsers wouldn't need definition lines inside SDP.
## E.g., "0" means G711 micro-law (PCMU) codec, "8" means G711 A-law (PCMA) codec. The browser automatically reads the static payload types and immediately knows what codec they are
## - Dynamic Payload Types (>=96): for modern high-fidelity codecs (e.g., Opus, VP8, ...) which created after the static numbers were filled, the browsers have no idea what '111' is until it reads a Codec Parameter line called `a=rtpmap` below
m=audio 58779 UDP/TLS/RTP/SAVPF 111 103 104 9 0 8 106 105 13 126

# 'c': connection line
## In ancient VoIP standard used for standard SIP landline calling, the connection line acts as the SSOT. However, Web applications running on different machine types and placed different network setup with complex routers, firewalls, NAT, ... A single static IP address will guarantee failure in public internet connection.
## Modern WebRTC standard just insert a dummy placeholder line or a masked local mDNS hostname (e.g., ff3034b4-0c1d-411e-8ff7-e07a86b79fca.local), the IP address is bypassed and unused
## Introduce ICE (Interactive Connectivity Establishment) protocol: the browser acts as an agent that gathers a large bundle of connection pathways for other machines to connect to the SDP creator, called "ICE Candidates". ICE is the chosen protocol to handle NAT traversal
##
c=IN IP4 0.0.0.0

# the identifier for 'a=group:BUNDLE 0 1', in case there are different media
a=mid:0
# define extensions used in RTP headers so receiver can decode it correctly and extract the metadata. E.g., audio level
a=extmap:1 urn:ietf:params:rtp-hdrext:ssrc-audio-level
# timestamp RTP packet
a=extmap:3 http://www.webrtc.org/experiments/rtp-hdrext/abs-send-time
# send+receive
a=sendrecv
# support multiplexing RTCP with RTP traffic
a=rtcp-mux

# ================================================ #
# Audio Lines => ICE Candidates                    #
# ================================================ #

# "Host candidate"
# Browser gives the private IP of the host machine as candidates
# Browser can receive/send SRTP and SRTCP on private IPv4 "192.168.0.196" if other peers happened on the same local network segment
# "2122260223": priority of the candidate. Usually's the most prioritized due to its directness have the best efficiency
a=candidate:1467250027 1 udp 2122260223 192.168.0.196 46243 typ host generation 0
# "Host candidate for RTCP on UDP
# Browser does not know if the other end supports rtcp-mux, therefore a port is needed in the offer
a=candidate:1467250027 2 udp 2122260222 192.168.0.196 56280 typ host generation 0

# Host candidates
a=candidate:435653019 1 tcp 1845501695 192.168.0.196 0 typ host tcptype active generation 0
a=candidate:435653019 2 tcp 1845501695 192.168.0.196 0 typ host tcptype active generation 0

# Server Reflexive candidates
# E.g., "47.61.61.61:36768" are public IP:port visible from STUN perspective
a=candidate:1853887674 1 udp 1518280447 47.61.61.61 36768 typ srflx raddr 192.168.0.196 rport 36768 generation 0
a=candidate:1853887674 2 udp 1518280447 47.61.61.61 36768 typ srflx raddr 192.168.0.196 rport 36768 generation 0

# Relay candidates
# Obtained from TURN server (e.g., coturn), which is provisioned when creating the RTCPeerConnection
# E.g., "237.30.30.30:51472" is the public IP:port assigned by the TURN server
# E.g., "47.61.61.61:54763" is the public-facing identity of the network router
a=candidate:750991856 2 udp 25108222 237.30.30.30 51472 typ relay raddr 47.61.61.61 rport 54763 generation 0
a=candidate:750991856 1 udp 25108223 237.30.30.30 58779 typ relay raddr 47.61.61.61 rport 54761 generation 0
...

# ================================================ #
# ICE Parameters                                   #
# ================================================ #

# For browsers reach each other via ICE candidates, authentication is required
a=ice-ufrag:Oyef7uvBlwafI3hT
a=ice-pwd:T0teqPLNQQOf+5W+ls+P2p16

# ================================================ #
# DTLS Parameters                                  #
# ================================================ #

# SHA-256 hash of the cert used in media encryption DTLS-SRTP negotiation
# Bind a trusted signaling and the cert used in TLS together, reject session if the fingerprint doesn't match
a=fingerprint:sha-256 49:66:12:17:0D:1C:91:AE:57:4C:C6:36:DD:D5:97:D2:7D:62:C9:9A:7F:B9:A3:F4:70:03:E7:43:91:73:23:5E
# this peer can be the client or the server which starts the DTLS negotiation (RFC4145), updated by RFC4572
a=setup:actpass

# ================================================ #
# Codec Parameters                                 #
# ================================================ #

# 'rtpmap:111': mapping payload type 111 to Opus audio codec.
# 'opus': one of the Mandatory-To-Implement audio codecs for WebRTC. Support VBR (6kbps-510kbps). Royalty-free => all major browsers support it.
# Translation: "Whenever you receive an RTP packet labeled with payload type "111", pass it to the Opus decoder running at 48kHz sample rate with 2 audio channels"
a=rtpmap:111 opus/48000/2

# configure the properties of Opus decoder, try to optimize according to best practices
a=fmtp:111 minptime=10; useinbandfec=1

# same as above
a=rtpmap:103 ISAC/16000
a=rtpmap:104 ISAC/32000
a=rtpmap:9 G722/8000
a=rtpmap:0 PCMU/8000
a=rtpmap:8 PCMA/8000
a=rtpmap:106 CN/32000
a=rtpmap:105 CN/16000
a=rtpmap:13 CN/8000
a=rtpmap:126 telephone-event/8000
# maximum amount of media that can be encapsulated in each packet
# unit: ms
# CAUTION: the size of the packet can have side effects in the quality of the audio and the BW.
a=maxptime:60

## SSRC parameters
a=ssrc:3570614608 cname:4TOk42mSjXCkVIa6
a=ssrc:3570614608 msid:lgsCFqt9kN2fVKw5wg3NKqGdATQoltEwOdMS 35429d94-5637-4686-9ecd-7d0622261ce8
a=ssrc:3570614608 mslabel:lgsCFqt9kN2fVKw5wg3NKqGdATQoltEwOdMS
a=ssrc:3570614608 label:35429d94-5637-4686-9ecd-7d0622261ce8

# ================================================ #
# Video Lines
# ================================================ #

m=video 60372 UDP/TLS/RTP/SAVPF 100 101 116 117 96
c=IN IP4 217.130.243.155
a=rtcp:64891 IN IP4 217.130.243.155

# ICE Candidates
a=candidate:1467250027 1 udp 2122260223 192.168.0.196 56143 typ host generation 0
a=candidate:1467250027 2 udp 2122260222 192.168.0.196 58874 typ host generation 0
a=candidate:435653019 1 tcp 1518280447 192.168.0.196 0 typ host tcptype active generation 0
a=candidate:435653019 2 tcp 1518280446 192.168.0.196 0 typ host tcptype active generation 0
a=candidate:1853887674 1 udp 1518280447 47.61.61.61 36768 typ srflx raddr 192.168.0.196 rport 36768 generation 0
a=candidate:1853887674 1 udp 1518280447 47.61.61.61 36768 typ srflx raddr 192.168.0.196 rport 36768 generation 0
a=candidate:750991856 1 udp 25108223 237.30.30.30 60372 typ relay raddr 47.61.61.61 rport 54765 generation 0
a=candidate:750991856 2 udp 25108222 237.30.30.30 64891 typ relay raddr 47.61.61.61 rport 54767 generation 0

# ICE Parameters
a=ice-ufrag:Oyef7uvBlwafI3hT
a=ice-pwd:T0teqPLNQQOf+5W+ls+P2p16

# DTLS Parameters
a=fingerprint:sha-256 49:66:12:17:0D:1C:91:AE:57:4C:C6:36:DD:D5:97:D2:7D:62:C9:9A:7F:B9:A3:F4:70:03:E7:43:91:73:23:5E
a=setup:actpass

a=mid:1
a=extmap:2 urn:ietf:params:rtp-hdrext:toffset
a=extmap:3 http://www.webrtc.org/experiments/rtp-hdrext/abs-send-time
a=extmap:4 urn:3gpp:video-orientation
a=sendrecv
a=rtcp-mux

# Codec Parameters

# "When you receive a RTP packet with payload type 100, send it to VP8 decoder"
a=rtpmap:100 VP8/90000
a=rtcp-fb:100 ccm fir
a=rtcp-fb:100 nack
a=rtcp-fb:100 nack pli
# (legacy) utilize REMB for bandwidth estimation
a=rtcp-fb:100 goog-remb
# "When you receive a RTP packet with payload type 101, send it to VP9 decoder"
a=rtpmap:101 VP9/90000
a=rtcp-fb:101 ccm fir
a=rtcp-fb:101 nack
a=rtcp-fb:101 nack pli
# (legacy) utilize REMB for bandwidth estimation
a=rtcp-fb:101 goog-remb
a=rtpmap:116 red/90000
a=rtpmap:117 ulpfec/90000
a=rtpmap:96 rtx/90000
a=fmtp:96 apt=100

# SSRC Parameters
a=ssrc-group:FID 2231627014 632943048
a=ssrc:2231627014 cname:4TOk42mSjXCkVIa6
a=ssrc:2231627014 msid:lgsCFqt9kN2fVKw5wg3NKqGdATQoltEwOdMS daed9400-d0dd-4db3-b949-422499e96e2d
a=ssrc:2231627014 mslabel:lgsCFqt9kN2fVKw5wg3NKqGdATQoltEwOdMS
a=ssrc:2231627014 label:daed9400-d0dd-4db3-b949-422499e96e2d
a=ssrc:632943048 cname:4TOk42mSjXCkVIa6
a=ssrc:632943048 msid:lgsCFqt9kN2fVKw5wg3NKqGdATQoltEwOdMS daed9400-d0dd-4db3-b949-422499e96e2d
```

2. VoIP SDP (Cre: MDN)

```sdp
# Protocol version
v=0
# Originator
# "A user named 'alice' created this session, along with her session ID, network type (IN), address type (IP4), and host address"
o=alice 2890844526 2890844526 IN IP4 host.example.com
# Session name
# can be left blank
s=
# Connection: where creators of the SDP expect to receive data. What kind of data?
# - creators as subscribers: media (video, audio) from the publishers
# - creators as publishers: network statistics, quality feedback (RTCP packets) from the subscribers
c=IN IP4 host.anywhere.com
# Attribute: Direction (recvonly, sendonly, sendrecv)
a=recvonly
# "The session is permanent or unbounded since it has no start or bound time"
t=0 0

# Media description/media lines/media streams

# 1st audio track
# Alice: "I've opened port 49170 on my machine, please send your audio packets to this port using RTP protocol, payload type 0"
# Bob: "I've received Alice's SDP Offer, I've generated my own SDP Answer containing my opened ports for Alice to send me RTCP packets. I will sent audio stream to Alice's IP address on port 49170 and I will prepare to receive RTCP packets from Alice as well."
m=audio 49170 RTP/AVP 0
# map payload type 0 to PCMU audio codec, sample rate 8000Hz
a=rtpmap:0 PCMU/8000

# 1st video track, received on port 51372 using RTP protocol, payload type 31
m=video 51372 RTP/AVP 31
# map payload type 31 to H261 video codec, clock rate 90000Hz
a=rtpmap:31 H261/90000

# 2nd video track, received on port 53000 using RTP protocol, payload type 32
m=video 53000 RTP/AVP 32
# map payload type 32 to MPV (MPEG Video) codec
a=rtpmap:32 MPV/90000
```
