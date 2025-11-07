# Docker Network

There are many types of Docker network drivers that engineers should know.

## Overlay

Enable containers running on different Docker hosts to communicate securely as if they were on the same local network.

Technically speaking, it uses VXLAN tunneling to encapsulate network traffic between nodes.

Overlay network needs either Docker Swarm and Docker Enterprise. All participating nodes must be on the same cluster.
