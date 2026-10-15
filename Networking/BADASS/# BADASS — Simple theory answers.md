Based on the questions in [42.EvalHub.pdf](42.EvalHub.pdf) and [the subject](en.subject.pdf).

## Project overview

### How does GNS3 work?

GNS3 lets us build virtual network topologies and connect devices such as routers and Docker containers. We can configure these devices and capture traffic between them.

### What does BGP do, and why is it useful?

BGP exchanges reachability information between routers and selects routes according to routing policies. It connects networks on the Internet, and its EVPN extension distributes Ethernet reachability in our project.

### What is the difference between Layer 2 and Layer 3?

Layer 2 forwards Ethernet frames using MAC addresses within a local network. Layer 3 routes IP packets between networks using IP addresses.

## Part 1 — Routing software and services

### What is packet routing software?

Routing software learns routes and updates the operating system's routing table so packets can reach their destinations. We use FRRouting, also called FRR.

### What is BGPD?

`bgpd` is the FRR service that runs BGP and exchanges reachability information with other routers. In Part 3, it distributes EVPN routes.

### What is OSPFD?

`ospfd` is the FRR service that runs OSPF, a protocol that learns paths within a network. In Part 3, it makes the routers' loopback addresses reachable.

### What is the IS-IS routing engine service?

`isisd` runs IS-IS, a routing protocol that learns the network topology and calculates shortest paths. It is enabled in our router images but is not used by the Part 3 topology.

### What is BusyBox?

BusyBox combines many common Linux commands into one small executable. It provides basic tools such as a shell, `ls`, and `ping` while keeping our Alpine containers lightweight.

## Part 2 — VXLAN

### What is VXLAN, and how does it differ from a VLAN?

A VLAN separates an Ethernet network into logical segments. VXLAN carries Ethernet frames inside UDP/IP packets, allowing a logical Ethernet segment to extend across an IP network.

### What is a switch?

A switch connects devices within an Ethernet network and forwards frames using a table of MAC addresses. It learns those addresses from the source MAC of incoming frames.

### What is a bridge?

A bridge joins network interfaces into the same Ethernet network and acts like a software switch. Our `br0` connects the local host-facing interface to the VXLAN interface.

### What is the difference between broadcast and multicast?

Broadcast targets every device in a broadcast domain. Multicast targets devices subscribed to a particular group.

### How should the Part 2 topology work?

Each router bridges its local host to VXLAN VNI 10, so the hosts communicate as if they were on the same Ethernet network. The static version specifies the remote tunnel endpoint, while the multicast version uses a shared multicast group to distribute flooded traffic.

## Part 3 — BGP EVPN

### What is BGP EVPN?

BGP EVPN uses BGP to advertise Ethernet reachability, such as which VTEP owns a host's MAC address. VXLAN carries the actual Ethernet frames between those VTEPs.

### What is route reflection?

A route reflector redistributes iBGP routes between its clients, so every leaf does not need a BGP session with every other leaf. Our spine router R1 acts as the route reflector.

### What does VTEP mean?

VTEP means VXLAN Tunnel Endpoint: it adds VXLAN headers to outgoing Ethernet frames and removes them from incoming tunnel traffic. Our three leaf routers are VTEPs.

### What does VNI mean?

VNI means VXLAN Network Identifier and identifies a virtual Ethernet segment. We use VNI 10 on all three VTEPs so their attached hosts share the same segment.

### What is the difference between type-2 and type-3 routes?

A type-2 route advertises a host's MAC address, optionally with an IP address. A type-3 route advertises a VTEP's participation in the segment so broadcast and other flooded frames can reach it, even when no hosts are running.

### How should the Part 3 topology work?

OSPF makes the routers reachable, and the route reflector distributes EVPN routes between the leaves. The leaves carry host Ethernet traffic through VXLAN VNI 10, while the spine forwards the outer IP packets.
