# WHIP/WHEP

Standardized signaling protocols for WebRTC

1. Signaling Transport Protocol:

+ Protocol: HTTP/HTTPS
+ Methods: POST for initial SDP offer/answer; PATCH for sending incremental ICE candidates; DELETE for stream teardown.
+ Flow control: HTTP status code (201 Created, 204 No Content) and HTTP headers (`Location:` header to pass back a unique session ID)

2. Signaling Data Format Protocol:

+ Session Setup: HTTP POST Request Body must be MIME type `application/sdp` and define strict constraints on that SDP.
+ ICE Trickle: HTTP PATCH must be MIME type `application/trickle-ice-sdpfrag`, a strict text data format for candidate exchange that MediaMTX `reader.js` script handles.
