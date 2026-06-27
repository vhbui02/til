# WebRTC in HTTP Proxy

HTTP proxy can't carry UDP packets because they're built exclusively for TCP using HTTP `CONNECT` method.

## Solutions

1. MASQUE protocol (RFC9298) allows tunneling UDP datagrams over HTTP. But it's new, experimental and lack of supports

2. TURN-over-TCP

- Make sure ICE servers setup with `?transport=tcp`
- Check inside WebRTC internal dump: 
+ no `onicecandidateerror`
+ `onicegatheringstatechange` is `gathering` => `complete`.
+ `onsignalingstatechange`: new => `have-local-offer` => `stable`.
