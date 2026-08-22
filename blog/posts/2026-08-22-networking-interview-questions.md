# Networking Interview Questions & Answers

A solid, interview-ready reference for **computer networking** — the OSI/TCP-IP models,
IP addressing, core protocols, routing/switching, DNS, HTTP, TLS, and troubleshooting.
Grouped by theme, with answers concise enough to say aloud but complete enough to defend
a follow-up. Useful for DevOps, cloud, backend, and SRE interviews.

---

## Models & fundamentals

**1. What is the OSI model?**

A 7-layer conceptual framework for how data moves through a network:

1. **Physical** — bits over the medium (cables, radio).
2. **Data Link** — framing, MAC addressing (Ethernet, switches).
3. **Network** — logical addressing and routing (IP, routers).
4. **Transport** — end-to-end delivery (TCP, UDP).
5. **Session** — establishing/managing sessions.
6. **Presentation** — encryption, encoding, compression.
7. **Application** — user-facing protocols (HTTP, DNS, SMTP).

**2. OSI vs. TCP/IP model?**

The TCP/IP model has 4 layers: **Link (Network Access)**, **Internet**, **Transport**,
**Application** — mapping the OSI's 7 layers into a practical, protocol-driven stack that
the internet actually uses.

**3. What is encapsulation?**

As data moves down the stack, each layer wraps it with its own header (and sometimes
trailer): segment → packet → frame → bits. The reverse (decapsulation) happens on receipt.

**4. What's the difference between a hub, switch, and router?**

- **Hub** (L1) — dumb repeater; broadcasts to all ports (obsolete).
- **Switch** (L2) — forwards frames by MAC address to the correct port.
- **Router** (L3) — forwards packets between networks by IP, choosing paths.

**5. MAC address vs. IP address?**

A **MAC** address is a hardware identifier (L2), globally unique, fixed to a NIC. An
**IP** address is a logical, routable address (L3) that can change by network. MAC is
local delivery within a segment; IP is end-to-end across networks.

**6. What is ARP?**

Address Resolution Protocol maps a known **IP** address to its **MAC** address on a local
network, so a device can build the L2 frame. Cached in an ARP table.

**7. What is a broadcast, unicast, and multicast?**

**Unicast** — one-to-one. **Broadcast** — one-to-all on a segment. **Multicast** —
one-to-many (a subscribed group). Anycast — to the nearest of many (used by CDNs/DNS).

---

## IP addressing & subnetting

**8. IPv4 vs. IPv6?**

IPv4 is 32-bit (~4.3 billion addresses), written as dotted decimal (`192.168.1.1`).
IPv6 is 128-bit (vast space), written in hex groups (`2001:db8::1`), with built-in
features and no need for NAT. IPv6 solves IPv4 exhaustion.

**9. What is a subnet mask / CIDR?**

A subnet mask (or CIDR prefix like `/24`) divides an IP into **network** and **host**
portions. `192.168.1.0/24` means the first 24 bits are network, leaving 8 bits (254
usable hosts). CIDR replaced classful addressing for flexible allocation.

**10. How many hosts in a `/24`, `/26`, `/30`?**

`/24` → 256 addresses, 254 usable. `/26` → 64, 62 usable. `/30` → 4, 2 usable (point-to-point
links). Usable = 2^(host bits) − 2 (network + broadcast addresses).

**11. What are private IP ranges?**

Defined by RFC 1918: `10.0.0.0/8`, `172.16.0.0/12`, `192.168.0.0/16`. Not routable on the
public internet; used inside LANs/VPCs with NAT for outbound access.

**12. What is NAT?**

Network Address Translation maps private IPs to a public IP (and back), letting many
internal hosts share one public address. **PAT/NAPT** (overload) uses port numbers to
multiplex many hosts behind one IP.

**13. What is a default gateway?**

The router a host sends packets to when the destination is outside its local subnet — the
exit point to other networks/the internet.

**14. What is DHCP?**

Dynamic Host Configuration Protocol automatically assigns hosts an IP, subnet mask,
gateway, and DNS servers via a lease. The DORA process: **Discover, Offer, Request,
Acknowledge.**

**15. What is a VLAN?**

A Virtual LAN logically segments a physical switch into isolated broadcast domains,
improving security and reducing broadcast traffic. Devices on different VLANs need a
router/L3 switch to communicate.

**16. What is subnetting and why do it?**

Splitting a network into smaller subnets to improve performance (smaller broadcast
domains), security (isolation), and address-space efficiency, and to map to organizational/topology boundaries.

---

## Transport layer

**17. TCP vs. UDP?**

**TCP** is connection-oriented, reliable, ordered, with flow/congestion control and
retransmission (web, email, file transfer). **UDP** is connectionless, unreliable, low
overhead, faster (DNS, VoIP, streaming, gaming). Choose TCP for correctness, UDP for
speed/tolerance to loss.

**18. Explain the TCP three-way handshake.**

To establish a connection: client sends **SYN**, server replies **SYN-ACK**, client
sends **ACK**. Sequence numbers are exchanged to synchronize both sides before data flows.

**19. How does TCP terminate a connection?**

The four-way handshake: each side sends a **FIN** and the other **ACK**s it (FIN, ACK,
FIN, ACK), closing each direction independently. `TIME_WAIT` ensures late packets drain.

**20. What are TCP flags?**

Control bits: **SYN** (synchronize), **ACK** (acknowledge), **FIN** (finish), **RST**
(reset/abort), **PSH** (push), **URG** (urgent). They manage connection state.

**21. What is flow control vs. congestion control?**

**Flow control** (sliding window) prevents a fast sender from overwhelming a slow
receiver. **Congestion control** (slow start, congestion avoidance, etc.) prevents
overwhelming the **network**, adjusting the send rate to observed loss/latency.

**22. What is a port number? Name some well-known ports.**

A 16-bit identifier for a service/process on a host. Well-known: 22 (SSH), 25 (SMTP), 53
(DNS), 80 (HTTP), 123 (NTP), 443 (HTTPS), 3306 (MySQL), 5432 (PostgreSQL), 6379 (Redis).

**23. What is a socket?**

The combination of an IP address and a port (plus protocol) that uniquely identifies one
endpoint of a connection. A TCP connection is defined by the 4-tuple (src IP:port, dst
IP:port).

**24. What is MTU and fragmentation?**

MTU (Maximum Transmission Unit) is the largest frame size a link can carry (typically
1500 bytes for Ethernet). Packets larger than the path MTU are fragmented (IPv4) or
dropped with a "too big" message (IPv6/PMTUD).

---

## Routing & switching

**25. What is routing?**

The process of selecting paths for packets across networks. Routers use a **routing
table** (destination prefix → next hop) built statically or via routing protocols.

**26. Static vs. dynamic routing?**

**Static** routes are manually configured (simple, no overhead, no auto-recovery).
**Dynamic** routing protocols (OSPF, BGP, EIGRP) automatically learn and adapt routes as
topology changes.

**27. What is BGP?**

Border Gateway Protocol is the routing protocol of the **internet** — an
exterior/path-vector protocol that exchanges reachability between autonomous systems
(ASes) and makes policy-based path decisions.

**28. OSPF vs. BGP?**

**OSPF** is an interior gateway protocol (within an organization/AS), link-state,
fast-converging, metric-based. **BGP** is an exterior protocol between ASes, path-vector,
policy-driven, and scales to the whole internet.

**29. What is the difference between L2 and L3 switching?**

**L2 switching** forwards frames by MAC within a broadcast domain. **L3 switching**
(a switch that can route) forwards by IP between VLANs/subnets at near-wire speed.

**30. What is a routing table / how does a router pick a route?**

It matches the destination IP against known prefixes using **longest prefix match**
(most specific wins), then forwards to that route's next hop/interface.

**31. What is spanning tree protocol (STP)?**

An L2 protocol that prevents switching loops by disabling redundant paths, building a
loop-free tree, and re-enabling links if the active path fails.

---

## DNS

**32. What is DNS?**

The Domain Name System translates human-readable domain names into IP addresses (and
other records). It's a hierarchical, distributed database — the "phone book" of the internet.

**33. Walk through a DNS resolution.**

A client asks its **recursive resolver**, which (if not cached) queries the **root**
servers → **TLD** servers (e.g. `.com`) → the domain's **authoritative** name server,
returning the record, which the resolver caches per its **TTL**.

**34. What are common DNS record types?**

- **A** / **AAAA** — name → IPv4 / IPv6.
- **CNAME** — alias to another name.
- **MX** — mail servers.
- **TXT** — arbitrary text (SPF, DKIM, verification).
- **NS** — authoritative name servers.
- **PTR** — reverse lookup (IP → name).
- **SOA** — zone authority info.
- **SRV** — service location.

**35. Recursive vs. iterative query?**

In a **recursive** query, the resolver does all the work and returns the final answer. In
an **iterative** query, each server returns a referral to the next server and the client/resolver follows the chain.

**36. What is TTL in DNS?**

Time To Live — how long a record may be cached before re-querying. Lower TTL = faster
propagation of changes but more query load; higher TTL = better caching, slower changes.

**37. A vs. CNAME record — key difference?**

An **A** record points a name directly to an IP. A **CNAME** points a name to **another
name** (alias). You can't have a CNAME at the zone apex (root domain) alongside other
records (use ALIAS/ANAME or an A record there).

---

## Application layer, HTTP & TLS

**38. What happens when you type a URL and press Enter?**

DNS resolves the hostname → TCP connection (three-way handshake) to the server IP:port →
(HTTPS) TLS handshake → HTTP request sent → server responds → browser renders, fetching
further assets. Caching, redirects, and CDNs may short-circuit steps.

**39. HTTP vs. HTTPS?**

HTTP is plaintext (port 80). HTTPS is HTTP over **TLS** (port 443), providing encryption,
integrity, and server authentication via certificates — protecting against eavesdropping
and tampering.

**40. What are common HTTP methods?**

`GET` (read), `POST` (create/submit), `PUT` (replace), `PATCH` (partial update),
`DELETE` (remove), `HEAD` (headers only), `OPTIONS` (capabilities/CORS preflight).

**41. What do HTTP status code classes mean?**

- **1xx** informational.
- **2xx** success (200 OK, 201 Created, 204 No Content).
- **3xx** redirection (301 permanent, 302 found, 304 not modified).
- **4xx** client error (400, 401, 403, 404, 429).
- **5xx** server error (500, 502, 503, 504).

**42. What's the difference between 502, 503, and 504?**

**502 Bad Gateway** — an upstream gave an invalid response. **503 Service Unavailable** —
server overloaded/down (often intentional). **504 Gateway Timeout** — an upstream didn't
respond in time. Common in load-balancer/proxy setups.

**43. What is TLS and how does the handshake work (briefly)?**

TLS secures connections via a handshake: negotiate versions/ciphers, the server presents
a **certificate** (authenticated via a CA chain), keys are exchanged (e.g. ECDHE for
forward secrecy), and both derive a shared symmetric session key for encryption.

**44. What is a digital certificate / CA?**

A certificate binds a public key to an identity (domain), signed by a trusted
**Certificate Authority**. Clients trust the chain up to a root CA in their trust store,
verifying the server's identity.

**45. HTTP/1.1 vs. HTTP/2 vs. HTTP/3?**

**HTTP/1.1** — one request at a time per connection (head-of-line blocking). **HTTP/2** —
multiplexed streams over one TCP connection, header compression. **HTTP/3** — runs over
**QUIC (UDP)**, eliminating TCP head-of-line blocking with faster connection setup.

**46. What is a cookie vs. a session?**

A **cookie** is client-side data the browser stores and sends back per request. A
**session** is server-side state (often keyed by a session ID stored in a cookie) that
tracks a user across requests.

**47. What is CORS?**

Cross-Origin Resource Sharing — a browser security mechanism where a server uses HTTP
headers (`Access-Control-Allow-Origin`, etc.) to declare which other origins may access
its resources, relaxing the same-origin policy safely.

---

## Load balancing, proxies & CDNs

**48. What is a load balancer?**

A device/service that distributes incoming traffic across multiple backend servers to
improve availability, scalability, and performance, using health checks to route only to
healthy backends.

**49. L4 vs. L7 load balancing?**

**L4** balances by IP/port (TCP/UDP) — fast, protocol-agnostic. **L7** balances by
application data (HTTP paths, headers, cookies) — smarter routing, TLS termination,
content-based rules — at higher cost.

**50. Common load-balancing algorithms?**

Round-robin, weighted round-robin, least connections, least response time, IP hash
(sticky by client), and consistent hashing.

**51. Forward proxy vs. reverse proxy?**

A **forward proxy** sits in front of **clients** (outbound; used for filtering,
anonymity, caching). A **reverse proxy** sits in front of **servers** (inbound; used for
load balancing, TLS termination, caching, WAF) — e.g. nginx, HAProxy.

**52. What is a CDN?**

A Content Delivery Network caches content on edge servers geographically close to users,
reducing latency and origin load, and absorbing traffic spikes/DDoS. Often uses anycast
routing.

**53. What is sticky session / session affinity?**

Routing a given client consistently to the same backend (via cookie or IP hash) so
server-side session state stays valid — needed when sessions aren't shared/externalized.

---

## Security & VPN

**54. What is a firewall?**

A device/software that filters traffic based on rules (IP, port, protocol, state).
**Stateful** firewalls track connection state; **stateless** filter per-packet. It
enforces the network security boundary.

**55. Stateful vs. stateless firewall?**

A **stateful** firewall remembers established connections and allows return traffic
automatically. A **stateless** one evaluates each packet independently against rules
(faster, less context).

**56. What is a VPN?**

A Virtual Private Network creates an encrypted tunnel over a public network, so remote
users/sites securely access a private network as if local. Protocols: IPsec, WireGuard,
OpenVPN.

**57. What is a DMZ?**

A "demilitarized zone" — a perimeter subnet exposing public-facing services (web, mail)
isolated from the internal network, so a compromise there doesn't directly reach internal systems.

**58. What is a DDoS attack and mitigation?**

A Distributed Denial of Service floods a target with traffic from many sources.
Mitigations: CDNs/scrubbing services, rate limiting, anycast, over-provisioning, and
upstream filtering (e.g. Cloudflare, AWS Shield).

**59. Symmetric vs. asymmetric encryption?**

**Symmetric** uses one shared key (fast; for bulk data). **Asymmetric** uses a public/private
key pair (slower; for key exchange and signatures). TLS uses asymmetric to exchange a
symmetric session key.

---

## Troubleshooting & tools

**60. How do you troubleshoot "can't reach a website"?**

Layer by layer: check local connectivity (`ping` gateway), DNS (`nslookup`/`dig`),
routing (`traceroute`), the port (`telnet`/`nc host 443`), TLS/cert, and the app
response (`curl -v`). Isolate whether it's DNS, network, or the service.

**61. What does `ping` do?**

Sends ICMP echo requests to test reachability and measure round-trip latency/packet loss.
No reply may mean the host is down **or** ICMP is filtered.

**62. What does `traceroute`/`tracert` do?**

Maps the path to a destination by sending packets with increasing TTL, revealing each hop
and its latency — useful to locate where connectivity breaks or slows.

**63. What is `dig`/`nslookup` used for?**

Querying DNS to resolve names and inspect records/TTLs, test specific name servers, and
debug resolution issues.

**64. What do `netstat`/`ss` show?**

Active connections, listening sockets, and their states (`LISTEN`, `ESTABLISHED`,
`TIME_WAIT`) — useful to confirm a service is listening or spot connection buildup.

**65. What is packet capture (tcpdump/Wireshark)?**

Capturing and inspecting raw packets to analyze protocol behavior, latency, retransmissions,
handshakes, and errors at the wire level — the ultimate ground truth for network debugging.

**66. `curl -v` — what does it reveal?**

Verbose HTTP(S) request/response including DNS resolution, TCP connect, TLS handshake and
certificate details, request/response headers, and status — invaluable for API/web debugging.

**67. What is latency vs. bandwidth vs. throughput?**

**Latency** is delay (time for a packet to travel, RTT). **Bandwidth** is the maximum
capacity of a link. **Throughput** is the actual achieved data rate. High bandwidth
doesn't fix high latency.

---

## Quick-fire round

- **Reliable transport?** TCP; fast/unreliable → UDP.
- **Name → IP?** DNS (A/AAAA record).
- **IP → MAC on a LAN?** ARP.
- **Auto IP assignment?** DHCP.
- **HTTPS port?** 443; SSH → 22; DNS → 53.
- **Private ranges?** 10/8, 172.16/12, 192.168/16.
- **Longest prefix match?** How routers choose a route.
- **Internet's routing protocol?** BGP.
- **L7 vs L4 LB?** App-aware vs. IP/port.
- **Test path to a host?** `traceroute`.
- **Encrypted tunnel over public net?** VPN.

---

These questions cover the ground most networking-focused interviews walk — from the OSI
model and subnetting through TCP handshakes, DNS resolution, TLS, and hands-on
troubleshooting. To make them stick, practice subnetting on paper, run `dig`/`curl -v`/`traceroute`
against real sites and read the output, and capture a TCP handshake in Wireshark. Seeing
the packets once turns these from memorized facts into intuition.
