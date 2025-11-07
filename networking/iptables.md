# `iptables`

Network configuration has always been seen as black magic, even among the tech workers.

## Workflow

Here is the packet processing workflow:

1. A network packet arrives at a network interface (e.g., `eth0`, `wlp1s0`, etc.).

1. The packet enters either the `PREROUTING` or `OUTPUT` chain, depending on its origin (external source or locally generated).

1. The packet is evaluated against each rule in the selected chain. Each rule specifies match criteria such as protocol, source/destination IP, source/destination port, subnet, interface, header, or connection state.

1. If a rule matches, its associated action (target) is executed (e.g., `ACCEPT`, `DROP` (silent discard), `REJECT` (explicit notify to the sender), `LOG`, etc.).

1. Actions are classified as either terminal (`ACCEPT`, `DROP`, `REJECT`) or non-terminal (`LOG`, `SNAT`, `DNAT`, `MASQUERADE`, etc.).

1. If the action is `ACCEPT`, the packet proceeds to the next chain. If `DROP`, the packet is discarded. If no rule matches, the chain's default policy is applied (typically `ACCEPT` or `DROP`).

1. The packet is then routed to the appropriate next chain:

- Packets from the internet to the local server: `PREROUTING → INPUT`.
- Packets from the local server to the internet: `OUTPUT → POSTROUTING`.
- Packets being routed through the server: `PREROUTING → FORWARD → POSTROUTING`.

- Packets destined for the local machine will enter the `INPUT` chain.
- Packets destined for other machines matching NAT rules in `PREROUTING` chain will enter `FORWARD` chain.

> [!IMPORTANT]
>
> If the packets is coming from Docker, and since Docker uses its own network namespace, the packets from containers may traverse the host's FORWARD chain or INPUT chain, depending on Docker's network mode. For inter-container and container-to-external traffic, Docker containers use the FORWARD chain. This is to say that `ufw` or `firewalld` firewall that acts as the manager `INPUT`/`OUTPUT` chain's rules, simply don't work with Docker network traffic.

1. Packet arrival: a network packet arrives at a network interface (e.g `eth0`, `wlp1s0`, ...)

1. Chain selection: the packet starts from either `PREROUTING` chain or `OUTPUT` chain, depends on its origin (external systems, or generated externally)

1. Rule matching:

- The packet is checked against each rule in the chain, each rule has a set of match criteria (e.g. protocol, src/dest IP, src/dest port, src/dest subnet, input/output interface, header, state...).

- If a rule matches, its action (or `target`) is applied (e.g. `ACCEPT, DROP (slient fail), REJECT (explicit fail), LOG, ...`).

4. Action Taken (more about this later):

Actions are categorized into 2 groups:

- Non-terminal actions: `LOG`, `SNAT`, `DNAT`, `MASQUERADE`, ...
- Terminal actions: `ACCEPT`, `REJECT` and `DROP`

> [!NOTE]
>
> The difference between `REJECT` and `DROP` is to use `DROP` if you DO NOT want the other end to know the port is unreachable, and use `REJECT` if you want to inform them.

If the action is:

- `ACCEPT`, the packet continues to the next chain.
- `DROP`, the packet is discarded.

If no rule matches, the chain's default policy is applied (usually either all `ACCEPT` or all `DROP`).

5. Next Chain Selection

A rule can be specified so package can jump to another chain.

- Packet from the internet to your server: `PREROUTING → INPUT`.
- Packet from your server to the internet: `OUTPUT → POSTROUTING`.
- Packet routed through your server: `PREROUTING → FORWARD → POSTROUTING`.

Packets that're destined to local machines (either originally, or after being NAT-ed after matching a `nat` rule in `PREROUTING`) will go into `INPUT` chain.

Packets that're destined to machines that's not the current machine (either originally, or after being NAT-ed after matching a `nat` rule in `PREROUTING`) will go into `FORWARD` chain.

> [!TIP]
>
> Docker create a custom chain called `DOCKER-USER`

## Documentation

### Overview

Linux used kernel-based packet filtering framework called _Netfilter_. Netfilter framework can use either of the THREE backend (or "iptables implementation"):

| Backend         | Kernel Subsystem | Technology | CLI tools            |
| --------------- | ---------------- | ---------- | -------------------- |
| iptables-legacy | xtables          | Legacy     | `iptables/ip6tables` |
| iptables-nft    | nf_tables        | Modern     | `iptables/ip6tables` |
| nftables        | nf_tables        | Modern     | `nft`                |

**Chain/Hooks:** intercepts the packet at a specific point. There are 5 built-in chains:

- `PREROUTING`
- `INPUT`
- `FORWARD`
- `OUTPUT`
- `POSTROUTING`

There can be custom chain as well, such as `DOCKER-USER`, `MAILCOW`, ... but they must be placed in the execution sequence of the built-in chains.

There are many types of rules inside a chain (a.k.a _table_):

- Most commonly known:`filter`, `nat`.
- Less commonly known: `mangle`, `raw`, `security`

Each rule have a set of criterias. Packets that matched all of the criterias in a rule will be applied to an _action_. There are two types of targets:

- Terminating : `-j DROP`, `-j ACCEPT`, `-j REJECT`.
- Non-terminating: `-j LOG`, `-j SNAT`, `-j DNAT`, `-j MASQUERADE`, ...

Finally, Network/System Administrators/Engineers write a rule with all chain, tables and targets under a single command line:

```sh
# accept any TCP packets that's destined to this machine's port 22, regardless of source IP
# In short: SSH
iptables -t filter -A INPUT -p tcp --dport 22 -j ACCEPT
```

### How to read the rules?

Run either of the below commands to list all of the rules inside `filter` table that're corresponding to a chain.

```sh
sudo iptables -L "${CHAIN_NAME}" -v -n
sudo iptables -L "${CHAIN_NAME}" -v --line-numbers | column -t
```

Example: `dockerized-mailcow`

```sh
Chain  INPUT  (policy                   DROP  127K  packets,  9872K  bytes)
pkts   bytes  target                    prot  opt   in        out    source    destination
344K   96M    MAILCOW                   all   --    any       any    anywhere  anywhere
159M   104G   ufw-before-logging-input  all   --    any       any    anywhere  anywhere
159M   104G   ufw-before-input          all   --    any       any    anywhere  anywhere
131M   48G    ufw-after-input           all   --    any       any    anywhere  anywhere
129M   48G    ufw-after-logging-input   all   --    any       any    anywhere  anywhere
129M   48G    ufw-reject-input          all   --    any       any    anywhere  anywhere
129M   48G    ufw-track-input           all   --    any       any    anywhere  anywhere

Chain  MAILCOW  (2      references)
pkts   bytes    target  prot         opt  in           out         source    destination
0      0        DROP    tcp          --   !br-mailcow  br-mailcow  anywhere  anywhere
```

> [!NOTE]
>
> There is no such thing as nested chains/hooks. Chains are sequentials. Rules can be specified for packet to jump from one chain to another.

- Policy: if not matched any of the rules, `DROP` by default.
- Top-rule matches: `MAILCOW`, then a sequence of `ufw` hooks.
- Interpretation: "Drops all TCP packets that're not coming from the `br-mailcow` interface, but destined for the `br-mailcow` interface, regardless of their source or destination IP addresses. If not, check them against the list of UFW custom chains to either `ACCEPT` or `REJECT`."

```sh
Chain  FORWARD  (policy  DROP                        0     packets,  0    bytes)
pkts     bytes    target                      prot  opt       in   out     source    destination
682K     181M     MAILCOW                     all   --        any  any     anywhere  anywhere
189M     86G      DOCKER-USER                 all   --        any  any     anywhere  anywhere
189M     86G      DOCKER-FORWARD              all   --        any  any     anywhere  anywhere
1        134      ufw-before-logging-forward  all   --        any  any     anywhere  anywhere
1        134      ufw-before-forward          all   --        any  any     anywhere  anywhere
0        0        ufw-after-forward           all   --        any  any     anywhere  anywhere
0        0        ufw-after-logging-forward   all   --        any  any     anywhere  anywhere
0        0        ufw-reject-forward          all   --        any  any     anywhere  anywhere
0        0        ufw-track-forward           all   --        any  any     anywhere  anywhere

Chain  DOCKER-USER  (1     references)
num    pkts         bytes  target       prot  opt  in   out  source    destination
1      470M         216G   RETURN       all   --   any  any  anywhere  anywhere

Chain  DOCKER-FORWARD  (1                        references)
pkts   bytes           target           prot         opt  in               out  source    destination
189M   86G             DOCKER-CT        all          --   any              any  anywhere  anywhere
37M    5928M           DOCKER-ISO...    all          --   any              any  anywhere  anywhere
37M    5927M           DOCKER-BRIDGE    all          --   any              any  anywhere  anywhere
457K   27M             ACCEPT           all          --   docker0          any  anywhere  anywhere
72636  4037K           ACCEPT           all          --   br-3cc34298ace3  any  anywhere  anywhere
613K   1828M           ACCEPT           all          --   br-10094c929d8d  any  anywhere  anywhere
93916  18M             ACCEPT           all          --   br-108b76218506  any  anywhere  anywhere
380K   92M             ACCEPT           all          --   br-520863aeee3b  any  anywhere  anywhere
1009   155K            ACCEPT           all          --   br-6f79277262ec  any  anywhere  anywhere
169    12684           ACCEPT           all          --   br-572d2659fa36  any  anywhere  anywhere
6269   1942K           ACCEPT           all          --   br-51ab7ea6ce79  any  anywhere  anywhere
754    85018           ACCEPT           all          --   br-9dcc13ae4fdd  any  anywhere  anywhere
157K   18M             ACCEPT           all          --   br-mailcow       any  anywhere  anywhere
```

=> Drop all packets that're destined to `br-mailcow` interface but're not coming from the `br-mailcow` itself. If not, check them against all rules in `DOCKER-USER`. However `DOCKER-USER` only contains the default `RETURN` rule, which renders the chain existance itself inside the `FORWARD` chain redundant. Then the traffic is being forwarded between Docker container interfaces. If not, check them against the list of UFW custom chains to either `ACCEPT` or `REJECT`. Finally, if none of the rules matching the packget, `DROP` it.

### Tables

#### `filter` tables

- Accept terminate actions/targets.
- `INPUT`, `FORWARD`, `OUTPUT` chains are allowed.
- This is the default if `-t` option is not specified.

```sh
# E.g. allow incoming connections on port 22 (SSH)
iptables -t filter -A INPUT -p tcp --dport 22 -j ACCEPT

# E.g. block incoming connections from a specific IP address
iptables -t filter -A INPUT -s 192.168.1.100 -j DROP
```

> [!NOTE]
>
> `-A` stands for "append", means appeading to the chain.

#### `nat` (Network Address Translation)

Any packets that match a rule inside `nat` table will have their Source/Destination IP/Port changed.

- Only `FORWARD` chain are not allowed. `PREROUTING`, `INPUT`, `OUTPUT`, `POSTROUTING` are okay.
- Only non-terminal actions are allowed.

```
Redirect incoming traffic to internal web server

Incoming Packet
      |
      v
PREROUTING (nat)  <- [DNAT]
(e.g. delegate to another machine by modifying IP address)
      |
      v
Routing decision  -> `ip route`, `ip rule`
      |
      v
   FORWARD
      |
      v
POSTROUTING (nat) <- [MASQUERADE] / [SNAT]
(e.g. change private IP to public IP)
      |
      v
Outgoing Packet
```

Show the rules in the `PREROUTING` chain in `iptables`:

```sh
sudo iptables -t nat -L PREROUTING -n
```

```sh
# after setting `net.ipv4.ip_forward=1`, the machine technically becomes a router
# it route the packets that're destined to this machine's port 80 to another machine
# whose private IP is 192.168.1.10 and port 80
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.10:80

# it "allows the traffic"
# of TCP packets that're destined to another machine whose IP is 192.168.1.10 and port 80
# to be forwarded through this machine
# this rule is for packets that're NOT originating from or destined to this machine
iptables -t filter -A FORWARD -p tcp --destination 192.168.1.10 --dport 80 -j ACCEPT

# ensures any packets belonging to established or related connections that are not originally from or destined to this machine, are allowed to pass through this machine
iptables -t filter -A FORWARD -m state --state ESTABLISHED,RELATED -j ACCEPT
```

```sh
# e.g. used when your machine acts as the router for other machines in a private network
# it "replaced the source IP address of any packets that leave the machine via eth0 interface with the router's public IP address"
# in other words, hiding IP address from the outside world
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE

# for normal connection, physical router has done this for you automatically
```

```sh
# Load balancing
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.10:80
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.11:80 -m statistic --mode nth --packet 0
```

```sh
# Forward web traffic to internal web server
iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 192.168.1.10:8080
```

```sh
# Redirect requests to another port
iptables -t nat -A PREROUTING -p tcp --dport 80 -j REDIRECT --to-port 8080
```

```sh
# Allows multiple devices on a internal/private network to share a single public IP address to access the internet
# Router have this rules by default
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j MASQUERADE

# MASQUERADE and SNAT are used to change the source of packets
# -j MASQUERADE is used when router public IP of eth0 is dynamic (typical for home internet connections)
# -j SNAT (or source NAT) is preferred when static IP is utilized.
iptables -t nat -A POSTROUTING -s 192.168.1.0/24 -o eth0 -j SNAT --to-source 203.0.113.5 # router public IP
```

#### `mangle`

- Change packet's header.
- Support 5 chains: `PREROUTING`, `INPUT`, `OUTPUT`, `FORWARD`, `POSTROUTING`.

```sh
# Mark packets from specific source
iptables -t mangle -A PREROUTING -s 192.168.1.0/24 -j MARK --set-mark 1
```

```sh
# Set TTL value for outgoing packets
iptables -t mangle -A POSTROUTING -o eth0 -j TTL --ttl-set 64
```

#### `raw`

- Less common.
- Configure exemptions from connection tracking.

#### `security`

- Less common
- Used with SELinux to manage security contexts.

### Chains

#### 1. `INPUT`

- Rules applied before packets went inside processes.
- Supported tables: `mangle`, `nat`

#### 2. `FORWARD`

- Rules applied for packets that are routed through the current host.
- Supported tables: `mangle`, `filter`

#### 3. `OUTPUT`

- Rules applied right after it's created from processes
- Supported tables: `mangle`, `nat`, `filter`, `raw`

#### 4. `PREROUTING`

- Rules applied as soon as they come in network interfaces.
- Supported tables: `mangle`, `nat`, `raw`.

#### 5. `POSTROUTING`

- Rules applied as they are about to go outside network interfaces.
- Supported tables: `magnel`, `nat`.
