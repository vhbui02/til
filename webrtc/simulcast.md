# [Simulcast](https://bloggeek.me/webrtcglossary/simulcast/)

## Procedure

1. Publishers encode a single video source into multiple streams simultaneously with different:
- resolutions
- bitrates
- frame rates
- overall quality

2. Streams are sent to WebRTC Media Server/SFU, which dynamically routes the most appropriate stream to each peer based on:

- Subscriber's available downlink bandwidth, decided through [bandwidth estimation](https://bloggeek.me/webrtcglossary/bwe/) techniques: REMB, transport-cc

 their network conditions, layout, device capabilities.

3. 

## Trade-offs

- Increase publisher device's CPU and uplink bandwidth.

