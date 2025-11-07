# `ip`

## `ip rule`

Analogy: "Traffic Director"

- Like receptionist directing people to different departments.
- Same as `iptables`, only more intuitive, network adminitrators create a set of _rules_ to determine **WHICH** routing table to use on packets that matched a set of criteria, such as src/dest IP, interface, mark, ...
- Multiple rules can match a packet, introduces the concept of "priority": the lower the number, the higher the priority. First matching rule is used.

```sh
# view all rules
ip rule show
ip rule list

# Default output usually looks like
# .
0:      from all lookup local       # local/broadcast address
32766:  from all lookup main        # most rules are inside here
32767:  from all lookup default     # empty

# Rule selectors
# Source-based
ip rule add from 192.168.1.0/24 lookup TABLE_NAME [priority] [NUMBER]

# Destination-based
ip rule add to 10.0.0.0/8 lookup TABLE_NAME [priority] [NUMBER]

# interface-based
ip rule add iif eth0 lookup TABLE_NAME [priority] [NUMBER]   # incoming
ip rule add oif eth0 lookup TABLE_NAME [priority] [NUMBER]   # outgoing

# firewall mark
ip rule add fwmark NUMBER lookup TABLE_NAME [priority] [NUMBER]
```

Troubleshooting:

```sh
ip rule show
ip route show table TABLE_NAME
ip route get DEST_IP from SRC_IP
```

Best practices:

- Keep rules minimal.
- Document the rules.
- Use meaningful table name.
- Consider table priority.
- Back up configuration: `ip rule show > rules.bak` and `ip route show > routes.bak`

## `ip route`

After knowing which routing table will be used for a specific packet, the _rules_ inside that table will decide _how that packet is handled_ and _where it will go_. `ip route` rule uses CIDR notation to indicate a particular packet destination.

Rules from `iptables` and `ip rule` has done a great job in matching the desired packets. but usually not all packets in a routing table are handled the same so you will need _rules_ for `ip route` as well. Its matching mechanism is very much the same as `iptables` and `ip rule`.

### Route selectors

```sh
# default route: applies this rule to all packets if none of other rules matched
# forward the packet to gateway 192.168.1.1
ip route add default via 192.168.1.1

# network route
# forward the packet to gateway 192.168.1.1 if destination IP matches 10.0.0.0/24 subnet
ip route add 10.0.0.0/24 via 192.168.1.1

# host route
# forward the packet to gateway 192.168.1.1 if destination IP matches 10.0.0.1/32
ip route add 10.0.0.1/32 via 192.168.1.1

# interface route
# delegate packet handling to network interfaces
ip route add 192.168.2.0/24 dev eth0
```

### Additional info

```sh
# multiple routing rules can be matched
# propose the "metric", the lower the number, the higher the priority
ip route add 10.0.0.0/24 via 192.168.1.1 metric 100

# if "table" option is not specified, it's added to table `main`
# best practice: policy-based routing, create a custom table with higher priorty than main
ip route add 10.0.0.0/24 via 192.168.1.1 table CUSTOM_TABLE

# protocols (not layer 3 tcp or layer 7 http))
ip route add 10.0.0.0/24 via 192.168.1.1 proto [static|kernel|dhcp|...]
```

ECMP (Equal-Cost Multi-Path, use for load-balancng):

```sh
ip route add default \
    nexthop via 192.168.1.1 dev eth0 weight 1 \
    nexthop via 192.168.2.1 dev eth1 weight 1
```

Different packets denied expression:

```sh
# blackhole route (drop packets)
ip route add blacklist 10.0.0.0/24

# unreachable route (return ICMP unreachable)
ip route add unreachable 10.0.0.0/24

# prohibit route (return ICMP prohibited)
ip route add prohibit 10.0.0.0/24
```

Troubleshooting:

```sh
# test routing decision
ip route get 8.8.8.8
ip route get 8.8.8.8 from 192.168.1.100
```

```sh
# show 'main' routing table
ip route
ip route show table main   # verbose

default via 10.65.0.1 dev wlan0 proto dhcp src 10.65.0.11 metric 305
# forward the packets to a gateway with address 10.65.0.1 if none of other rules match

10.33.0.0/16 dev vpn1 scope link
# forward the packets to network interface vpn1 if the destination matches 10.33.0.0/16

10.65.0.0/20 dev wlan0 proto dhcp scope link metric 305
# forward the packets to network interface wlan0 if the destination matches 10.65.0.0/20
# no gatewaysince the packets' destination is in the same network as the gateway, so the packets will be sent directly.
```

By default, as shown in `ip route`, there are 3 routing tables which are **local**, **main** and **default**, each table has an ID (**not priority**) and a name.

## `ip link`

View, modify, configure _network interfaces_ on system.

### CRUD operations

```sh
# display all network interfaces
ip link show

# bring an interface up/down
ip link set dev eth0 [up|down]

# Change MAC address, Maximum Transmission Unit (MTU), ...
ip link set dev eth0 address 00:11:22:33:44:55
ip link set dev eth0 mtu 1400

# Setup Virtual Interface...
```

## References

- [blogd's "`iptables`chuyên sâu"](https://blogd.net/linux/iptables-chuyen-sau/)
