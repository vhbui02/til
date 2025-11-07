# TCP Proxy Vs HTTP Proxy Vs SOCKS5 Proxy

## The differences between TCP Proxy vs HTTP Proxy vs SOCKS5 Proxy

### 1. TCP Proxy

- **OSI Layer:** Layer 4 (Transport layer)
- **Description:** Forwards raw TCP packets between a client and a destination server.
- **Use Cases:**
  - Basic, fast load balancing for TCP-based applications
  - Port forwarding (e.g. forward from port 80 to 8000 for multi-domain hosting strategy)
  - Bypass IP-based firewalls.
  - Hiding the origin server's IP address from the client (and vice-versa)
  - Lower overhead than higher-level proxies (HTTP, SOCKS5, ...) if no DPI is involved.
- **Cons:** No application-level awareness

```
┌────────┐    TCP connection    ┌───────┐    TCP connection    ┌────────┐
│ Client │ ═══════════════════> │ Proxy │ ═══════════════════> │ Server │
└────────┘                      └───────┘                      └────────┘
  │                              │                              │
  │         TCP segments         │         TCP segments         │
  │<═══════════════════════════> │<═══════════════════════════> │
  │                              │                              │
```

1. Client establishes TCP connection to Proxy
2. Proxy establishes TCP connection to Server
3. Proxy relays TCP segments bidirectionally between Client and Server

### 2. HTTP Proxy

- **OSI Layer:** Layer 7 (Application Layer) [^1]
- **Description:** Understands HTTP protocol, interprets HTTP Requests and HTTP Responses to fulfill operations.
- **Use Cases:**
  - Web caching.
  - Content filtering (block access to APIs based on client's metadata)
  - Anonymity (hide the client's IP address from the destination server)
  - Access control and logging.
  - Modify HTTP headers
- **Cons:** Introduces more latency than a TCP proxy, due to parsing and processing application-layer data

**HTTP (insecure) Proxy:**

```
┌────────┐  HTTP Request   ┌───────┐  HTTP Request   ┌────────┐
│ Client │ ═══════════════>│ Proxy │ ═══════════════>│ Server │
│        │                 │       │  (parsed, check │        │
│        │                 │       │   modified?)    │        │
│        │  HTTP Response  │       │  HTTP Response  │        │
│        │<═══════════════ │       │<═══════════════ │        │
└────────┘  (not modified  └───────┘  (cached)       └────────┘
            return cache)
```

**HTTPS (secure) Proxy:**

```
┌────────┐ HTTP CONNECT    ┌───────┐    TCP tunnel    ┌────────┐
│ Client │ ═══════════════>│ Proxy │ ═══════════════> │ Server │
│        │                 │       │                  │        │
│        │    SSL handshake, direct communnication    │        │
│        │<══════════════════════════════════════════>|        │
└────────┘                 └───────┘                  └────────┘
```

Proxy won't be able to intercept HTTP Requests from clients anymore, unless it uses SSL termination, which is only feasible when:

- Clients install a certificate issued by the Proxy
- Proxy terminate SSL, inspect the content, modify, ... then finally re-encrypt and send to Dest.

This break E2E encryption model. Clients will see Proxy's certificate instead of Destination server's certificate.

### 3. SOCKS5 Proxy

- **OSI Layer:** Layer 5 (Session Layer)
- **Description:** Routes network packets between a client and a destination server. The latest version, SOCKS5, supports authentication, UDP, and IPv6.
- **Use Cases:**
  - Bypass firewall
  - Routing TCP/UDP traffic for any Layer 7 protocol
  - Enhanced anonymity (DNS resolution by proxy)
- **Cons:** Client app must support SOCKS protocol and introduces overhead due to the negotiation.

```
┌────────┐                 ┌───────┐                  ┌─────────────┐
│ Client │   SOCKS5        │ Proxy │  TCP/UDP relay   │ Destination │
│ (SOCKS │ ═══════════════>│       │ ═══════════════> │             │
│ aware) │   negotiation   │       │                  │             │
│        │                 │       │                  │             │
│        │   Application data relayed through proxy   │             │
│        │<══════════════════════════════════════════>|             │
└────────┘                 └───────┘                  └─────────────┘
```

1. Client app (SOCKS-aware) connects to Proxy.
2. Client negotiates an auth method with proxy (none, password-based, ...)
3. After auth, Client specify command (CONNECT for TCP, UDP ASSOCIATE for UDP), Destination server's IP address + port.
4. Proxy ===TCP connection/UDP relay==> Dest
5. Proxy relay traffic between Client and Dest.

- Proxy won't be able to interpret the application data being transmitted.
- Proxy can perform DNS resolution on proxy side, which hides client's IP and DNS. Dest can only see proxy's IP.

[^1]: [What is layer 7 of the Internet?](https://www.cloudflare.com/learning/ddos/what-is-layer-7/)

## The differences between Reverse Proxy vs Forward Proxy

| Topic      | Forward Proxy              | Reverse Proxy              |
| ---------- | -------------------------- | -------------------------- |
| Connection | Private => public IP space | Public => private IP space |
| Security   | Trust the client           | Trust the server           |
