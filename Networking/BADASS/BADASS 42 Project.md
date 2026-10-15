# Part 3 — Defense walkthrough

**Defense date:** October 16, 2026

**References:** [Subject, section IV.3](../tmp/en.subject.pdf) · [Evaluation sheet, Part 3](../tmp/42.EvalHub.pdf)

Follow the steps in order. Each step explains **why** you perform the action,
then gives the commands and expected results.

- **VM terminal:** run commands from the BADASS repository root.
- **R1–R4 / H1–H3:** run commands inside that node's **Auxiliary console**.
- **Wireshark filters:** enter them in the display-filter field, not a terminal.

## Node reference

| Alias | GNS3 node | Role / connection | Router loopback |
| --- | --- | --- | --- |
| R1 | `p3-router-ocassany-1-1` | Spine / route reflector | `1.1.1.1` |
| R2 | `p3-router-ocassany-2-1` | VTEP, connected to H1 | `1.1.1.2` |
| R3 | `p3-router-ocassany-3-1` | VTEP, connected to H2 | `1.1.1.3` |
| R4 | `p3-router-ocassany-4-1` | VTEP, connected to H3 | `1.1.1.4` |
| H1 | `p3-host-ocassany-1-1` | Host behind R2 | — |
| H2 | `p3-host-ocassany-2-1` | Host behind R3 | — |
| H3 | `p3-host-ocassany-3-1` | Host behind R4 | — |

## Step 1 — Build the images and check the GNS3 versions

> **Why:** We first prepare the software and images used by this exact submission. GNS3 creates the topology and connects the Docker nodes through virtual links. A Docker image is a template; each GNS3 node is a separate container using it.

**Run in the VM terminal:**

```bash
pacman -Q gns3-gui gns3-server
make -C P3
make -C P3 images
```

**Expected:** GUI and server 2.2.61; all seven image tags are present. Run this on the VM/Docker server used by GNS3. Rebuilding an image does not update existing containers, so use the fresh import in the next step.

The three host tags refer to one shared image built from `P3/_ocassany_host`.
The Makefile builds it once and assigns all three names expected by the archive.
Each host node is a separate container with its own MAC address and runtime IP.

## Step 2 — Show the source files, import the project, and explain the topology

> **Why:** The evaluators need to see where each part of the configuration comes from. FRR (FRRouting) is the routing software. zebra handles interface/kernel routing integration; ospfd runs OSPF; bgpd runs BGP. isisd is enabled but unused here. Routing forwards IP packets between networks; bridging forwards Ethernet frames using MAC addresses. This lab uses both.

Show in the editor:

- `P3/_ocassany-1` and `P3/_ocassany_host` — router/shared-host Dockerfiles.
- `P3/config/daemons` — enabled FRR services.
- `P3/config/_ocassany-1.conf` — RR OSPF/BGP configuration.
- `P3/config/_ocassany-2.conf` — leaf OSPF/BGP configuration.
- `P3/config/_ocassany-1.sh` — explicit RR address setup.
- `P3/config/_ocassany-2.sh` — explicit leaf/VXLAN/bridge setup.

**Run in the VM terminal:**

```bash
gns3 P3/P3.gns3project
```

Complete the import into a NEW project directory; keep all nodes stopped. Show the diagram: R1 connects to all three leaves, each leaf to one host. The archive references the Docker images built in Step 1. In each router image, its config/_ocassany-N.sh is installed as /configure.sh.

## Step 3 — Start only the four routers and inspect their initial state

> **Why:** We demonstrate the requirement that the images assign no IPs by default. A loopback is an interface independent of a physical link. 127.0.0.1 is the normal local-only address; our custom 1.1.1.x loopbacks are not assigned yet. A router ID identifies a routing process; it does not assign an interface IP.

**In GNS3:**

Start R1, R2, R3 and R4. Leave H1, H2 and H3 STOPPED. Right-click each router -> Auxiliary console. Use Auxiliary console for an interactive shell; ordinary Console is attached to the startup process, which waits in sleep infinity.

**Run on each router:**

```bash
ip -br address
ps -o pid,comm,args
vtysh -c 'show version'
```

**Expected:** Ethernet interfaces have no IPs; no 1.1.1.x loopback addresses. IPv6 autoconfiguration is disabled, so no IPv6 link-local addresses appear. zebra, bgpd, ospfd and isisd are running. vtysh is FRR's command-line interface.

## Step 4 — Apply the route reflector's configuration

> **Why:** The underlay is the routed IP network carrying traffic between VTEPs. We now deliberately configure its addresses, rather than assigning them automatically when the images start. The RR is also the transit router here.

**Run on R1:**

```bash
/configure.sh
ip -br address
```

**Expected:**

- lo   1.1.1.1/32
- eth0 10.1.1.1/30 -> R2
- eth1 10.1.1.5/30 -> R3
- eth2 10.1.1.9/30 -> R4

The script returns to the shell. It can be rerun; it does not keep sleeping.

## Step 5 — Configure the three leaves explicitly

> **Why:** A leaf connects a host to the fabric. A VTEP (VXLAN Tunnel Endpoint) adds or removes VXLAN headers. The three leaves are VTEPs; R1 is not a VTEP. A bridge is a software Ethernet switch connecting the host port to vxlan10.

**Run separately on R2, then R3, then R4:**

```bash
/configure.sh
ip -br address
```

**Expected:**

- R2: lo 1.1.1.2/32; eth0 10.1.1.2/30; host port eth1 in br0.
- R3: lo 1.1.1.3/32; eth1 10.1.1.6/30; host port eth0 in br0.
- R4: lo 1.1.1.4/32; eth2 10.1.1.10/30; host port eth0 in br0.

All three now have br0 and vxlan10. FRR detects the new interfaces/addresses. After any router restart, run /configure.sh again.

## Step 6 — Verify OSPF and reachability between router loopbacks

> **Why:** OSPF (Open Shortest Path First) is a link-state interior routing protocol. It exchanges topology information and computes paths inside our network. FULL means neighbors have synchronized their link-state databases. Convergence means the routers have finished learning usable routes.

**Run on each router:**

```bash
vtysh -c 'show ip ospf neighbor'
vtysh -c 'show ip route ospf'
```

Allow convergence, usually tens of seconds. Repeat until neighbors are FULL. Expected: three FULL neighbors at R1; one FULL neighbor at each leaf. Routes marked O are learned through OSPF. Do not continue while routes or adjacencies are missing; an initial 2-Way/DR election state can be temporary.

**Run on R4:**

```bash
ping -c 2 -W 2 1.1.1.1
ping -c 2 -W 2 1.1.1.2
ping -c 2 -W 2 1.1.1.3
```

**Expected:** 2/2 replies from each, proving reachability to the RR and other VTEPs. You can repeat from every router to all other loopbacks if requested.

## Step 7 — Show the BGP EVPN sessions and route reflector settings

> **Why:** BGP exchanges reachability information. iBGP connects routers in the same autonomous system (AS); all four routers use AS 1 here. EVPN (Ethernet VPN) is the BGP address family distributing Ethernet reachability. Route reflection distributes client routes to other clients, avoiding an iBGP full mesh. Dynamic peering lets R1 accept leaves from its configured range.

**Run on R1:**

```bash
vtysh -c 'show bgp l2vpn evpn summary'
cat /etc/frr/frr.conf
```

**Expected:** three established peers, 1.1.1.2, 1.1.1.3, 1.1.1.4. Dynamic peers are marked *. The summary should show three dynamic neighbors. In State/PfxRcd, an established peer normally displays a NUMBER of received prefixes. Idle/Active means that the session is not established yet. Point out bgp listen range, neighbor LEAF remote-as 1, and neighbor LEAF route-reflector-client inside address-family l2vpn evpn.

**Run on each leaf:**

```bash
vtysh -c 'show bgp l2vpn evpn summary'
```

**Expected:** one established EVPN peer, 1.1.1.1.

## Step 8 — Prove VNI 10 exists and only type-3 routes exist with all hosts off

> **Why:** VXLAN carries Ethernet frames inside UDP/IP across the underlay. The overlay is the virtual Ethernet network carried by those tunnels. VNI (VXLAN Network Identifier) identifies the segment: 10 in this project. Type 3 / IMET advertises a VTEP's membership for BUM traffic: broadcast, unknown unicast and multicast. Type 2 advertises a MAC, optionally with an IP.

**Run on each leaf:**

```bash
ip -d link show vxlan10
bridge link show
vtysh -c 'show evpn vni'
```

**Expected:** vxlan id 10, dstport 4789, nolearning, and two remote VTEPs. br0 connects the host-facing port and vxlan10. Learning on the VXLAN bridge port is disabled: remote host MACs must come from BGP EVPN.

**Run on R1 and each leaf:**

```bash
vtysh -c 'show bgp l2vpn evpn route type multicast'
vtysh -c 'show bgp l2vpn evpn route type macip'
```

**Expected:** three type-3 prefixes; NO type-2 prefixes. multicast and macip are FRR's command keywords for types 3 and 2. A type-3 route does not require configuring a multicast VXLAN group here.

## Step 9 — Start only host 1, prove it has no IP, and generate Ethernet traffic

> **Why:** MAC learning is a Layer 2 operation: a bridge learns the source MAC of a frame on the interface where it arrives. An IPv4 address is not needed for this. ARP normally resolves IPv4 addresses to MACs. An ARP DAD probe can send a frame with sender IP 0.0.0.0, without assigning an IP to the sending host.

**In GNS3:**

Start H1 only; keep H2 and H3 stopped. Open H1's Auxiliary console. Wait for startup to finish and for eth0 to be UP before sending probes.

**Run on H1:**

```bash
ip link show dev eth0
ip address show dev eth0
arping -D -I eth0 -c 3 20.1.1.254
ip address show dev eth0
```

**Expected:** eth0 UP; no inet or inet6 address before or after the probes. Note its link/ether MAC address for the next step. arping reports sender 0.0.0.0. 20.1.1.254 is only a probe target, not an address assigned to H1. No response is expected because that target is unused. A silent host does not necessarily generate a learned route immediately.

## Step 10 — Show that the MAC-only type-2 route reaches the other leaves

> **Why:** EVPN's control plane advertises the learned MAC through the RR. The control plane exchanges reachability; the data plane forwards real frames. A remote MAC entry marked extern_learn is installed by an external control plane such as EVPN, rather than learned from VXLAN data packets.

**Run on R1 and each leaf:**

```bash
vtysh -c 'show bgp l2vpn evpn route type macip'
```

**Run on R2, R3 and R4:**

```bash
vtysh -c 'show evpn mac vni 10'
bridge fdb show dev vxlan10
```

**Expected:** H1's MAC appears as a type-2 route without a host IP. R2 shows the MAC local on eth1. R3/R4 show it remote through VTEP 1.1.1.2. R1 learns the route from R2 and reflects it to the other leaves. The remote next hop remains the originating VTEP, not R1's loopback. FDB means Forwarding Database: the bridge's MAC-to-port forwarding table.

## Step 11 — Start the remaining hosts and demonstrate their MAC learning too

> **Why:** Starting another host demonstrates that learning is dynamic, rather than a hardcoded mapping. We still keep host IPs unconfigured at this stage.

**In GNS3:**

Start H2 and H3. Open their Auxiliary consoles and wait for eth0 UP.

**Run separately on H2 and H3:**

```bash
ip address show dev eth0
arping -D -I eth0 -c 3 20.1.1.254
```

**Run on R1:**

```bash
vtysh -c 'show bgp l2vpn evpn route type macip'
```

**Expected:** three host MAC-only prefixes after all three hosts send traffic. H2 originates from VTEP 1.1.1.3; H3 originates from VTEP 1.1.1.4. No host has been assigned its overlay IPv4 address yet.

## Step 12 — Assign the three host addresses manually

> **Why:** We have proved MAC learning without IPs. Now we assign overlay addresses so the hosts can send ICMP pings. /24 means the first 24 address bits identify the subnet; all three hosts belong to 20.1.1.0/24 on the same virtual LAN.

**Run on H1:**

```bash
ip addr replace 20.1.1.1/24 dev eth0
```

**Run on H2:**

```bash
ip addr replace 20.1.1.2/24 dev eth0
```

**Run on H3:**

```bash
ip addr replace 20.1.1.3/24 dev eth0
```

**Run on each host:**

```bash
ip -br address
```

**Expected:** each host has only its own 20.1.1.x/24 address on eth0. No default gateway is needed for this same-subnet host-to-host test.

## Step 13 — Start a packet capture BEFORE launching the ping tests

> **Why:** Packet inspection is required: pings alone do not prove the encapsulation. We capture an underlay link, where both VXLAN traffic and OSPF are visible. ICMP (Internet Control Message Protocol) carries ping echo requests/replies.

**In GNS3:**

Right-click the R1 eth0 <-> R2 eth0 link -> Start capture. Open Wireshark and leave it capturing throughout the next step. Traffic from H1 to H2/H3 crosses this link. Let capture run at least 15–20 seconds to include periodic OSPF Hello packets as well as ping traffic.

Optional live terminal view instead of, or alongside, Wireshark:

**Run on R1 in a separate Auxiliary console; stop with Ctrl-C:**

```bash
tcpdump -nn -vv -i eth0 'ip proto 89 or udp port 4789'
```

## Step 14 — Ping between all three hosts while the capture is running

> **Why:** These tests exercise every directed host pair across different VTEPs. The leaves encapsulate/decapsulate the frames; R1 routes the outer IP packets.

**Run on H1:**

```bash
ping -c 3 -W 2 20.1.1.2
ping -c 3 -W 2 20.1.1.3
```

**Run on H2:**

```bash
ping -c 3 -W 2 20.1.1.1
ping -c 3 -W 2 20.1.1.3
```

**Run on H3:**

```bash
ping -c 3 -W 2 20.1.1.1
ping -c 3 -W 2 20.1.1.2
```

**Expected:** 3 transmitted, 3 received, 0% packet loss in every test. During initial ARP resolution, repeat a test if needed and investigate any continuing loss rather than presenting an unsuccessful result as a pass.

## Step 15 — Explain the captured VXLAN, ICMP and OSPF packets

> **Why:** Encapsulation adds outer headers while preserving the inner Ethernet frame. Outer IPs identify the VTEPs; inner IPs identify the communicating hosts. R1 is a transit router and a route reflector; it does not terminate VXLAN.

**Wireshark display filters — examine one at a time:**

```text
vxlan
vxlan && icmp
vxlan.vni == 10
ospf
```

Expand one H1 -> H2 ping packet:

- Outer source/destination: 1.1.1.2 -> 1.1.1.3.
- UDP destination port: 4789.
- VXLAN VNI: 10.
- Inner source/destination: 20.1.1.1 -> 20.1.1.2.
- Inner payload: ICMP echo request; find the reverse echo reply.

With the ospf filter, show Hello packets between 10.1.1.1/10.1.1.2 and multicast destination 224.0.0.5. OSPF uses IP protocol 89, not UDP port 89.

Say the complete path:

- H1 -> R2 bridge -> VXLAN over the underlay through R1 -> R3 -> H2.
- OSPF makes VTEPs reachable, EVPN distributes MAC reachability, and VXLAN carries the frames. The configuration does not use MPLS.

## Step 16 — Finish the demonstration and export the stopped project if requested

> **Why:** A portable export records the topology and project files for another import. Learned MACs are dynamic state, not permanent configuration. Stopping hosts may leave type-2 routes until MAC aging removes them (about 300 seconds here).

**In GNS3:**

Stop capture and save it if the evaluators want evidence. Stop all seven nodes before File -> Export portable project. If asked to repeat the type-3-only demonstration, restart the routers with hosts stopped, run /configure.sh on each again, and wait for convergence. Alternatively, allow the old host MAC entries to age before inspecting routes.

Remember: the images start without predefined addresses. Router configuration must be reapplied after restart, and host addresses are assigned only at the ping stage. A fresh machine needs the referenced Docker images built/loaded.

## Troubleshooting

- **/configure.sh missing:** likely an old container. Rebuild and use a fresh import.
- **eth0 down on a newly started host:** wait for its startup command to finish.
- **OSPF not FULL:** check scripts ran, link wiring, interface IPs and convergence.
- **BGP Idle/Active:** first verify router-loopback reachability through OSPF.
- **No type-2 route:** generate host traffic; inspect the local leaf MAC table first.
- **Unexpected type-2 routes with hosts off:** stale learned MACs may still be aging.
- **Ping fails:** verify host /24 addresses, EVPN peers, MAC entries and VNI 10.
- **Ordinary Console echoes commands:** use Auxiliary console instead.

## Validation of the updated images

Rebuilt all seven image tags with make -C P3, then imported P3/P3.gns3project into a temporary GNS3 2.2.61 project using fresh containers.

- [x] no predefined Ethernet addresses or custom loopback addresses on all seven nodes after startup; no per-node start-command overrides.
- [x] zebra, bgpd, ospfd, and isisd running on all four routers before explicit network setup; no VXLAN interface created automatically.
- [x] /configure.sh executed twice per router (eight successful runs).
- [x] all three RR OSPF neighbors FULL, all three dynamic EVPN peers established, VNI 10 active, and hosts-off state has three type-3 prefixes and no type-2 prefixes on every router.
- [x] all twelve directed router-loopback ping paths (2/2 replies each).
- [x] host 1 has no IP before or after ARP probes from 0.0.0.0; its MAC-only type-2 route is visible on all four routers without any CMD override.
- [x] hosts 2/3 also start without IPs. After explicit address assignment, all six directed host ping paths pass (18/18 replies, zero loss).
- [x] temporary project removed; original open projects retain node states.

Diagnostic commands were executed through docker exec on the temporary nodes. Startup addresses were checked with ip -j address show; daemon processes with ps -o comm; interface readiness with ip -j link show eth0; convergence with show ip ospf neighbor and show bgp l2vpn evpn summary json. Traffic commands and route-table checks are listed in steps 6–14 above. This updated-image test did not repeat the earlier packet-capture or export/reimport audit. The first attempt sent an ARP probe before the restarted host's interface was up; the completed test waited for eth0 UP before sending traffic. Session test log: /tmp/badass-p3-no-default-ip-test.log.

The earlier report in `tmp/defense-part3.txt` describes the previous images; its automatic-address findings and temporary-start-command workaround are historical and do not describe this updated startup flow.

### Shared host Dockerfile verification

The three host Dockerfiles were consolidated into `P3/_ocassany_host` after
confirming that their executable Docker instructions were identical. The
Makefile builds that file once with all three original host image tags.

- [x] All three rebuilt tags have exactly the same image IDs as before the refactor.
- [x] Image filesystem layers and runtime configuration match the previously tested images.
- [x] The project archive, router Dockerfiles, and routing configuration files retain their pre-refactor hashes.
- [x] The README and walkthrough reference the shared Dockerfile.

The earlier runtime results apply to the identical rebuilt images. A second
network test was unnecessary for this Dockerfile-only consolidation.
