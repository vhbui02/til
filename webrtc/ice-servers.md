# ICE servers

Here is the data flow and logic flow with statically-configured ICE servers

When a WebRTC client application starts with statically-configured ICE servers (e.g., `new RTCPeerConnection({ iceServers: [{ urls: "turn:...", username: "foo", credential: "bar" }] })`), the browser’s internal ICE Agent executes the following step-by-step workflow:

```
┌─────────────────┐       ┌──────────────────────┐       ┌──────────────────────┐
│  Client Browser │       │ STUN / COTURN Server │       │       MediaMTX       │
└────────┬────────┘       └──────────┬───────────┘       └──────────┬───────────┘
         │                           │                              │
         │ 1. Local Host Candidates  │                              │
         │─────────────────────────► │                              │
         │  (Discovers 192.168.x.x)  │                              │
         │                           │                              │
         │ 2. STUN Binding Request   │                              │
         │──────────────────────────►│                              │
         │ 3. STUN Binding Response  │                              │
         │◄──────────────────────────│                              │
         │  (Discovers public srflx) │                              │
         │                           │                              │
         │ 4. TURN Allocate Req (Auth)                              │
         │──────────────────────────►│                              │
         │ 5. TURN Allocate Success  │                              │
         │◄──────────────────────────│                              │
         │  (Allocates relay socket) │                              │
         │                           │                              │
         │ 6. WHEP Signaling (SDP Offer + ICE Candidates)           │
         │─────────────────────────────────────────────────────────►│
         │ 7. WHEP Signaling Response (SDP Answer)                  │
         │◄─────────────────────────────────────────────────────────│
         │                           │                              │
         │ 8. ICE Connectivity Checks (STUN Pings to candidate pairs)
         │─────────────────────────────────────────────────────────►│ (Direct Host Path)
         │  - - - - OR - - - - - - - │ - - - - - - - - - - - - - - -│
         │──────────────────────────►│─────────────────────────────►│ (Relayed Path)
```

Step-by-step logic flow:

1. Local Candidate Discovery (host):

- The browser inspects the device's physical network adapters (Wi-Fi, Ethernet, VPN, ...).

- Generates host candidates (e.g., 192.168.1.50:52100).

2. STUN Address Discovery (srflx):

- The ICE agent sends a UDP STUN Binding Request to the STUN URL (stun:coturn.example.com:3478).

- COTURN inspects the incoming packet's source IP/port and sends back a STUN Binding Response containing XOR-MAPPED-ADDRESS (e.g., 203.0.113.45:61234).

- The browser generates a srflx (Server Reflexive) candidate representing its public IP/port assigned by the local router's NAT.

3. TURN Relay Allocation (relay):

- The ICE agent sends a TURN Allocate Request using the static username and credential.

- COTURN checks its user database (turnserver.conf or database) to authenticate the user.

- Upon successful authentication, COTURN assigns a dedicated relay port on the COTURN server (e.g., 198.51.100.10:54321).

- The browser generates a relay candidate pointing to COTURN’s assigned relay port.

4. Signaling Exchange:

- The gathered candidate list (host, srflx, relay) is included in the SDP Offer of the initial WHEP HTTP POST request, or sent via HTTP PATCH request according to Trickle ICE, to MediaMTX

5. Connectivity Checks (ICE Probing):

- The browser prioritizes candidate pairs (host > srflx > relay).

- It sends STUN Binding Request pings across candidate pairs simultaneously.

- Whichever valid path responds with the lowest latency/highest priority is nominated for actual video delivery.
