# [Bandwidth Estimation (BWE)](https://bloggeek.me/webrtcglossary/bwe/)

Part of WebRTC Quality of Service (QoS) tools.

## REMB (Receiver Estimated Maximum Bitrate)

- Subscriber-side protocol, older than TWCC.
- Publishers timestamps all media packets.
- As the packets are received, subscribers calculate packet loss and arrival times.
- Subscribers then make the decision about the available bandwidth and decide when to increase or decrease bitrate.
- Subscribers communicates the estimated bitrate back to the sender.
- look for: `a=rtcp-fb:102 goog-remb`

=> Effectiveness rely on the algorithm chosen by the subscribers.

## TWCC (Transport-Wide Congestion Control)

- Publisher-side protocol. More modern than REMB, introduced and used by Google.
- Publishers attach a global sequence number to all media packets and remembers the send times based on the sequence number
- Subscribers calculate the time differences between received media packets using the global sequence number for reference and periodically sends a compact RTCP feedback message back to the Publishers.
- Publishers calculate the optimal bandwidth.
- Look for: `a=rtcp-fb:102 transport-cc`.

Pros:
- per-packet feedback across both video and audio streams.
- prevent bufferbloat
- maintain LL more effectively than REMB

## MediaMTX adoptation



