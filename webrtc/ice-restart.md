# ICE Restart

- Objective: triggers a fresh ICE negotiation on an existing PeerConnection without tearding down the session.
- Defined in IETF RFC 8445
- WebRTC API: `RTCPeerConnection.restartIce()`

## When to use ICE Restart

- A peer switches from Wifi-to to Cellular (or vice versa) or between two Wi-Fi networks.
- # A NAT binding timed out on router and a new public IP address is assigned
- Triggers a fresh ICE negotiation on an existing PeerConnection without tearding down the session.
- WebRTC API: `RTCPeerConnection.restartIce()`
- Defined in IETF RFC 8445

## When does ICE Restart occur?

- A device switches from Wifi to Cellular (or vice versa) or between two Wi-Fi networks.
- A NAT binding times out on router and a new public IP address is assigned
- ICE connection state moves to `failed` or `disconnected`
- TURN allocation expires or the relay path breaks.

## ICE Restart benefits

- Avoid creating new PeerConnection from scratch which could affect UX and waste resources.
- ICE restart lets the session to recover without the user noticing anything more than a short freeze.

## How ICE restart works

1. One side generates a new SDP offer with new `ice-ufrag` and `ice-pwd` values. NOTE: credentials scope an ICE session.
2. Both peers gather ICE candidates from scratch: STUN reflexive candidates, TURN relay candidates, ...
3. Connectivity checks run against new candidate pairs.
4. Once a working pair is nominated, media switches to the new path.
5. The old ICE session is discarded.

## How to trigger ICE restart?

1. `pc.restartIce()`: fires `negotiationneeded` event, the next `createOffer()` call will include the restart flag `{ iceRestart: true }` automatically. Preferred if implementing Perfect Negotiation.
2. `pc.createOffer({ iceRestart: true })`.

Common pattern: listen for ICE state changes:

```
let restartTimer;
pc.oniceconnectionstatechange = () => {
  if (pc.iceConnectionState === 'failed') {
    pc.restartIce();
  } else if (pc.iceConnectionState === 'disconnected') {
    restartTimer = setTimeout(() => {
      if (pc.iceConnectionState === 'disconnected') {
        pc.restartIce();
      }
    }, 2000); // 2 seconds is heuristic, tune it
  } else {
    clearTimeout(restartTimer);
  }
};
```

## ICE restart vs Full Negotiation

- ICE restart: replace ICE credentials and candidates. Codecs, SDP media sections, DTLS handshake, SRTP keys stay the same. => tool for connectivity recovery
- Full Negotiation: i.e., calling `createOffer()` without `{ iceRestart: true }` can add/remove tracks, change codecs, renegotiate media parameters. => tool for media changes
