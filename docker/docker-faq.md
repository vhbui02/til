# Docker FAQ

## Why Docker must be run in root privileged?

- Docker daemon binds to a UNIX socket, not a TCP port. Since by default only `root` user owns the UNIX socket, other users must access it via `sudo`.
- If you don't want to skip `sudo` part, just create a UNIX group called `docker` (some Docker packages on many package managers create this group automatically after the installation of Docker Engine) and add users to it.
- HOWEVER, you must think twice before running any `docker ...` commands, since this group grants root-level privileges to the user.

## Named volume vs Bind mount in terms of performance

- For Windows and MacOS, bind mount is slower than named volume.
- On Linux, the performance difference between bind mounts and named volumes is minimal and generally negligible.

## Security

<!-- TODO: read https://docs.docker.com/engine/security/#docker-daemon-attack-surface -->

## Firewall

- When exposing container ports using Docker, these ports bypass firewall rules (`ufw`/`firewalld`/...)
- Docker is only compatible with 2 Netfilter backends: `iptables-nft` and `iptables-legacy`. A more recent `nft` is not supported.
- Use `iptables`/`ip6tables` or `nftables` CLI tools to write firewall rulesets and add them to `DOCKER-USER` chain.

### `ufw`

If the packets is coming from Docker, and since Docker uses its own network namespace, the packets from containers may traverse the host's FORWARD chain or INPUT chain, depending on Docker's network mode. For inter-container and container-to-external traffic, Docker containers use the FORWARD chain.

=> This is to say that `ufw` or `firewalld` firewalls that act as the manager/wrapper of the `INPUT` + `OUTPUT` chain's rules, simply don't work with Docker network traffic.

### `firewalld`

If:

- Docker is running with `iptables: true` when running `docker info`
- `firewalld` is enabled.

1. Docker creates a zone called _docker_. All network interfaces created by Docker (e.g. by default - `docker0`) are inserted into `docker` zone.

2. Docker also creates a forwarding policy called `docker-forwarding` that allows forwarding from `ANY` zone to the `docker` zone.

If you want to control the incoming/outgoing traffic flow for Docker containers, you need to write `firewalld` rules targeting the `docker` zone.

```sh
# allow HTTP traffic to containers
firewall-cmd --zone=docker --add-service=http
firewall-cmd --zone=docker --add-service=https
```

> Check by running `firewall-cmd --get-active-zones`.

## CLI

```sh
# show the occupied disk space
docker system df

# clear build cache
docker builder prune [-f]
```

## References

- https://docs.docker.com/engine/network/packet-filtering-firewalls/#integration-with-firewalld
- https://docs.docker.com/engine/network/packet-filtering-firewalls/#docker-and-ufw
- https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user
