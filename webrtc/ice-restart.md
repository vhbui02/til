# ICE Restart

- Objective: triggers a fresh ICE negotiation on an existing PeerConnection without tearding down the session.
- Defined in IETF RFC 8445
- WebRTC API: `RTCPeerConnection.restartIce()`

## When to use ICE Restart

- A peer switches from Wifi-to to Cellular (or vice versa) or between two Wi-Fi networks.
- A NAT binding timed out on router and a new public IP address is assigned
- ICE connection state moves to `failed` or `disconnected`
- TURN allocation expires or the relay path breaks.

## ICE Restart benefits

- Avoid creating new PeerConnection from scratch which could affect UX and waste resources.
- ICE restart lets the session to recover without the user noticing anything more than a short freeze.

## 
