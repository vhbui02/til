# Third-party libraries

## 1. x186k/whip-whap-js

Cons:
- incorrect connection-state policy: `failed`, `closed` and `disconnected` all results in ICE restart.
+ `disconnected`: Wi-Fi interruption, Laptop roaming between APs, short packet-loss burtst, mobile network handover, browser tab/OS scheduling delay. Can be transitioned to `connected` or `completed`.
+ `

## 2. medooze/whip-whep-js

Pros:
- closer to a complete WHEP client
- store the resolved session URL from `Location` header (NOTE: session URL is different from endpoint URL) of the HTTP Response containing SDP answer and uses it for `PATCH` and `DELETE` requests.

> WHEP endpoint URL: sending HTTP Request to this `https://media.example.com/camera1/whep` creates sessions:
> ```
> POST /camera1/whep
> Content-Type: application/sdp
> 
> v=0
> ```
>
> WHEP session URL: the server responds
> ```
> HTTP/1.1 201 Created
> Location: /camera1/whep/sessions/7f4a
> Content-Type: application/sdp
> 
> v=0
> ```

- reject response that does not contain `Location:`

- proper ICE Trickle, `PATCH` HTTP request use MIME type `application/trickle-ice-sdp-frag`, sent to the session URL using `PATCH`.

- emits a `a=end-of-candidates`.

- includes `If-Match` handling and distinguishes ordinary ICE candidate updates from ICE restarts.
- in-session ICE restarts: preserve WHEP session, only replace ICE credentials and candidates via `PATCH`, no renegotiating non-ICE SDP.
- not responsible for the following => more reusable
+ transceivers
+ codec preferences
+ data channels
+ browser-specific constraints
+ `iceTransportPolicy`
- expose `this.onOffer = offer => offer` and `this.onAnswer = answer => answer`
- broader `Link` parsing, aligning with the current WHEP specification.
- `stop()` API closes the peer connection and sends `DELETE` to the WHEP session URL.

Cons:
- mismatch type declaration => fixable
- ICE configuration typo
- fragile SDP rewriting during ICE restart. only the concept is worth borrowing.
- weak request serialization, WHEP explicitly permits overlapping `PATCH` requests. ETags required when ICE restart is supported.
- introduce layer selection and SSE relationship URNs.
- poor EventSource authentication.

## 3. blueenviron/mediamtx

Pros:
- short (676 lines, 2026-07-20)
- best MediaMTX compatibility: maintained and evolved with MediaMTX.
- extensive codec compaibility logic: detects and conditionally injects codecs that browsers may support without adverising normally
- complete initial connection flow:

1. browser codec capability
2. ICE server discovery
3. `RTCPeerConnection` construction.
4. Transceiver setup: receive-only video and audio
5. Data-channel negotiation.
6. SDP offer construction and editing.
7. Offer parsing for ICE and media information
8. Send authenticated (Basic + Bearer) WHEP POST.
9. Session URL resolution.
10. Remote answer application.
11. Candidate queue flushing.

- complete full reconnection, trade simple and operationally robust over micro performance.

1. close old peer connection
2. send `DELETE`.
3. clear state
4. wait 2 seconds
5. create a new WHEP session

- HTML `read_index.html` provides useful browser behavior, supports URL-controlled:

1. Controls (Play, Pause, Fast-forward, ...)
2. Muting
3. Autoplay
4. Inline playback
5. Picture-in-picture disabling

Cons:

- Normal `close()` does not send `DELETE`:
1. Closes the browser peer connection
2. Cancels the retry timer
3. DOES NOT delete an active WHEP HTTP session.
=> Fixable 

- `DELETE` only sent from the error-recovery path.

- No in-session ICE Restart. => Fixable after PR 5770 supports server-side ICE Restart, a client-side implementation now is capable. Full reconnection as fallback.

- No `ETag` state.

- ICE servers are only read from `OPTIONS`, where the specification permits the endpoint to return ICE server links in the initial `201 Created` response. => Fixable, reading both responses will improve interoperability without MediaMTX.

- Assume `Link` value are seperated by `, ` (comma-with-space), and assumes a specific parameter order where a valid RFC 8288 Link field can vary in spacing, quoting, parameter ordering, and repeated field representation. => Fixable by creating a more generalize parser.

- Fixed reconnection interval: every failure retries after 2 seconds, no:
1. exponential backoff
2. jitter
3. Retry-After handling
4. Maximum retries
5. Distinction between transient and permanent failures.

- A missing stream causes indefinite requests, each fired every 2 seconds => DDOS your own server.

- No cancellation of in-flight HTTP operations: `close()` does not cancel outstanding `OPTIONS`, `POST`, `PATCH`, answer processing. => Fixable with `AbortController`

- Limited public lifecycle observability: 
1. `onError`
2. `onTrack`
3. `onDataChannel`

- No expose:
1. Connecting
2. Connected
3. Reconnecting
4. Closed
5. Session URL created
6. ICE restart attempted
7. Full reconnect attempted
- 
