# Telemetry

Traditional BE observability:

- Socket connection state
- ICE completion events
- SFU ingress/egress bitrates

=> Necessary, not enough. Represents infra view, not clients' view.

Green "server" coexists with "red" user experience. The server knows it forwards a packet but it does not know if: the packet arrived, in the correct order, or if the client's CPU/GPU decoder is able to render it.

Packet loss, buffer underruns, high jitter, ... are all client-side problems.

Types of problems:
- Network failure
- Device limitation
- Platform bug

=> Treat the client as active telemetry node in observability mesh.

## W3C Report Model - `RTCPeerConnection.getStats()`

Reports linked to each other.


```
const stats = await pc.getStats(); // RTCStatsReport (i.e., an iterable of RTCStats)
stats.forEach((stat) => {
  console.log(stat); // RTCStats

  // inbound-rtp: receiver POV metadata (packet loss, bytes received, decode metrics, ...)
  // outbound-rtp: sender POV metadata (bytes sent, encode metrics, ...)
  // remote-inbound-rtp: RTCP RR/Receiver Reports (RTT, fraction lost, ...) sent from receiver => sender
  // candidate-pair: accurate RTT, throughput capacity, ...
  // transport: DTLS state, ...
  console.log(stat.type);

  if (stat.type === "candidate-pair") {
    console.log(stat.currentRoundTripTime); // RTT
  }

  if (stat.type === "remote-inbound-rtp") {
    console.log(stat.roundTripTime); // RTT, calculated from RTCP RR reports
  }

  if (stat.type === "inbound-rtp") {
    // trade-off between interactivity (LL) and playback smoothness
    // using UDP as transport layer means packet can arrive out-of-order (e.g., p1 ~ 30ms, p2 ~ 90ms, p3 ~ 35ms)
    // one cannot play the packet as soon as it's arrived, it needs to be stored inside a temporary holding queue => jitter buffer
    // if the packet arrives after its scheduled playback back slot has already passed, the packet is useless => jitter buffer dropped it
    console.log(stat.jitter); // Jitter

    // WebRTC tolerate minor loss (5% packets) via NACKs + Forward Error Correction
    console.log(stat.packetsLost); // cumulative counter, calculate delta between samples to determine the current loss rate
  }

  // if you encountered video freeze, look into these stats firsthand
  console.log(stat.framesDecoded); // cumulative # of frames passed through decoder
  console.log(stat.framesDropped); // cumulative # of frames received but discarded (due to CPU saturation or buffer underruns)
  console.log(stat.bytesReceived);
});
```

A tip on video freeze detection: If `bytesReceived` is increasing (network is flowing) but `framesDecoded` delta is zero, the user is looking at a frozen frame and packets are being buffered.

- Byte stop: network cutoff
- Byte received yet video frozen: decoder failure, keyframe starvation.

## Client-side Normalization Architecture

- Poll `getStats()` 1-5s.
- Store the previous report, calculate `currentValue - previousValue` for cumulative counters such as `bytesReceived`, `packetsLoss`, `framesDecoded`
- Normalization: Convert raw bytes to bits-per-second (bps). Convert cumulative lost to lost-per-interval
- Batching: Send 5-10 samples at once before sending to the server to reduce HTTP/WebSocket overhead.

## Common Behaviors

1. Robotic Audio: dropped Opus audio packets are synthesized using Packet Loss Consealment algorithm. A few packets won't cause a significant change but multiple packets will create a metallic, choppy, "robotic" sound.

2. Frozen Video: Video compression algorkithm relies on Reference Frames (P-frames depending on prior I-frames/other P-frames). A late P-frame would be dropped, the video recorder cannot reconstruct that frame. The decoder stops rendering new frames and holds the last completed image until a full keyframe arrived.

## References

- [The Client Knows Best: Deep Dive into WebRTC getStats() and Quality Monitoring](https://dev.to/deepak_mishra_35863517037/the-client-knows-best-deep-dive-into-webrtc-getstats-and-quality-monitoring-3e5p)
