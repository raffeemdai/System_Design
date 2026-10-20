# System Design Interview Notes — Story-Style, Easy to Understand

> **How to use these notes:** Each topic has the same flow: **Story → Concepts → Analogy → Where it's used → Important points → Interview Q&A.**
> Read the story first. If the story makes sense, the concept will stick, and you can explain it in an interview in your own words.

## Table of Contents

**Core topics (your list)**
1. System design thinking & client-server fundamentals
2. IP addresses, ports & network fundamentals
3. DNS resolution & traffic routing
4. TCP, UDP, TLS & connection behavior
5. Proxies, CDNs & the edge
6. Load balancing & fault tolerance
7. RESTful APIs & HTTP contracts
8. API gateway, authentication & rate limiting
9. CORS & browser security
10. REST, GraphQL, gRPC & real-time communication
11. Caching patterns & invalidation
12. Scalability: horizontal, vertical, diagonal
13. Autoscaling & cloud operational practices
14. Performance engineering: latency, throughput & monitoring
15. Putting it together: product catalog & checkout

**Missing topics I added (they come up in almost every interview)**
16. Back-of-the-envelope estimation
17. Databases: SQL vs NoSQL, indexing
18. Replication, sharding & consistent hashing
19. CAP theorem, consistency models & ACID/BASE
20. Message queues & asynchronous processing
21. Microservices vs monolith & resilience patterns
22. Idempotency & distributed transactions (Saga)
23. Object storage, search & unique ID generation
24. Security essentials (HTTPS, OAuth, JWT, common attacks)
25. Observability: logs, metrics, traces, SLI/SLO/SLA
26. Disaster recovery, availability math & multi-region
27. The interview framework & final cheat sheet

---

# 01. System Design Thinking & Client-Server Fundamentals

## 📖 Story
Imagine you open a small tea stall. In week one, 20 customers come. You make tea, take money, hand over cups. Easy. Then a college opens nearby and 2,000 people show up. Now you have queues, spilled tea, wrong change, and angry customers.

You now need to *design*: more stoves, a cashier separate from the tea maker, a menu board, a backup gas cylinder. **System design is just this: planning how a business (software) keeps working smoothly when it grows from 20 customers to 20 million.**

In software, the customer is the **client** (your phone/browser) and the tea stall is the **server**. The customer *asks* ("one tea please" = **request**), the stall *answers* ("here you go" = **response**).

## 🧠 Concepts

**Client-Server model**
- **Client**: the one who asks (browser, mobile app, another service).
- **Server**: the one who answers (a computer running your code).
- Communication is **request → response**. The server doesn't talk first (in basic HTTP).

**Typical 3-tier architecture**
```
Client (Browser/App)  →  Application Server (business logic)  →  Database (data storage)
```

**Stateless vs Stateful**
- *Stateless server*: remembers nothing between requests. Every request carries all info needed (like a vending machine: you press, it gives).
- *Stateful server*: remembers you (like a waiter who knows "your usual order").
- Stateless is easier to scale because any server can serve any request.

**Functional vs Non-Functional Requirements**
- **Functional** = *what* the system does ("user can add item to cart").
- **Non-functional** = *how well* it does it: scalability, availability, latency, consistency, durability, security, cost.

**Core system design building blocks**: DNS, CDN, Load balancer, API gateway, App servers, Cache, Database, Queue, Object storage, Monitoring.

**Thinking process (the "design muscle")**
1. Clarify requirements (ask questions!).
2. Estimate scale (users, requests/sec, data size).
3. Define APIs & data model.
4. Draw high-level design.
5. Deep-dive into bottlenecks.
6. Discuss trade-offs, failures, and improvements.

## 🎯 Analogy
A restaurant: **Client** = customer, **Server** = waiter + kitchen, **Database** = storeroom, **Load balancer** = the host who decides which table/waiter you go to, **Cache** = the ready-made items on the counter.

## 🛠 Where it's used
Everything — Instagram feed, Uber ride request, Amazon checkout. All are client-server at the core.

## ⭐ Important Points
- There's **no single perfect design**; every design is a set of **trade-offs**.
- Always start with **requirements**, never with boxes and arrows.
- Prefer **stateless** app servers; push state into DB/cache.
- Single Point of Failure (**SPOF**) = anything whose failure kills the whole system. Hunt them.
- "Design for failure": hardware, network, and humans *will* fail.

## ❓ Interview Q&A

**Q1. What is system design?**
A: It's deciding how components (clients, servers, databases, caches, queues) fit together so that the system meets requirements like scale, speed, reliability, and cost. It's like planning the layout of a growing restaurant instead of just cooking food.

**Q2. Why is stateless design preferred?**
A: If a server stores no user session in memory, any server can handle any request. So I can add/remove servers freely and a crash loses nothing. Session data goes to Redis or a DB, or into a token like JWT.

**Q3. What's a single point of failure and how do you remove it?**
A: A component that, if it fails, takes down everything (one DB, one load balancer). Remove it with redundancy: replicas, multiple load balancers, multi-AZ deployment, and automatic failover.

**Q4. Functional vs non-functional requirements — give an example.**
A: Functional: "User can place an order." Non-functional: "Order placement should respond under 300 ms for 99% of requests and the system should be 99.9% available."

**Q5. What do you do first in a design interview?**
A: Ask clarifying questions: who are the users, what features are in scope, expected scale, read-heavy or write-heavy, consistency needs. Then estimate, then design.

**Q6. Monolith vs distributed system — why go distributed?**
A: One machine has limits (CPU, RAM, disk, and it's a SPOF). Distributing gives scale and fault tolerance but adds complexity: network failures, data consistency, debugging difficulty.

---

# 02. IP Addresses, Ports & Network Fundamentals

## 📖 Story
Think of a huge apartment building. The **building's street address** is the **IP address**. Each **flat number** is a **port**. A courier needs both: the street address gets them to the building, the flat number gets them to the right family.

Your laptop has an IP. Inside it, the browser might use one flat, Spotify another, and a web server a third. The OS uses **ports** to hand the right data to the right program.

## 🧠 Concepts

**IP address**: a unique number identifying a device on a network.
- **IPv4**: `192.168.1.10` (32-bit, ~4.3 billion addresses, running out).
- **IPv6**: `2001:0db8::1` (128-bit, practically unlimited).
- **Public IP**: reachable from the internet. **Private IP**: only inside a local network (`10.x.x.x`, `172.16–31.x.x`, `192.168.x.x`).
- **NAT (Network Address Translation)**: many devices at home share one public IP; the router translates. It's like a company reception: one public phone number, many internal extensions.

**Ports**: numbers 0–65535.
| Port | Used for |
|---|---|
| 80 | HTTP |
| 443 | HTTPS |
| 22 | SSH |
| 53 | DNS |
| 3306 | MySQL |
| 5432 | PostgreSQL |
| 6379 | Redis |
| 27017 | MongoDB |

**Socket** = IP + Port (+ protocol). A connection is identified by `(client IP, client port, server IP, server port)`.

**CIDR / Subnets**: `10.0.0.0/24` means the first 24 bits are the network part, leaving 8 bits (256 addresses) for hosts. Subnets split a big network into smaller ones (public subnet, private subnet).

**Network layers (OSI simplified / TCP/IP model)**
```
Application  (HTTP, DNS, SMTP)      ← what apps speak
Transport    (TCP, UDP)             ← reliable or fast delivery, ports
Network      (IP)                   ← addressing & routing
Link/Physical(Ethernet, Wi-Fi)      ← actual wires/signals
```
Remember: **L4** = TCP/UDP level, **L7** = HTTP level (matters for load balancers later).

**Firewall / Security Groups**: rules that say "allow port 443 from anywhere, port 22 only from office IP, DB port only from app servers."

**Bandwidth vs Latency**: Bandwidth = width of the pipe (MB/s). Latency = time for one drop to travel (ms). A truck full of hard disks has huge bandwidth but terrible latency.

## 🎯 Analogy
IP = building address, Port = flat number, Router = the postal sorting hub, Firewall = security guard at the gate, NAT = reception desk of a company.

## 🛠 Where it's used
Cloud VPCs, private subnets for databases, security groups, microservice communication (`10.0.2.15:8080`), Kubernetes pod networking.

## ⭐ Important Points
- Databases should live in **private subnets**, never directly exposed to the internet.
- Only the load balancer/API gateway should have public exposure.
- IPv4 exhaustion is why NAT and IPv6 exist.
- Network is **not reliable**: packets get dropped, delayed, duplicated, reordered.
- Latency is limited by the speed of light: Mumbai → New York can't be faster than ~100+ ms round trip. Put data closer to users.

## ❓ Interview Q&A

**Q1. Difference between IP address and port?**
A: IP finds the machine; port finds the application on that machine.

**Q2. What is NAT and why is it used?**
A: It lets many private devices share one public IP by translating addresses at the router. It saves IPv4 addresses and hides internal structure.

**Q3. Public vs private subnet?**
A: Public subnet has a route to the internet gateway (load balancers, bastion hosts). Private subnet has no direct inbound internet access (app servers, databases). Private ones can reach out via a NAT gateway.

**Q4. What is a CIDR block like /16 vs /24?**
A: /16 = 65,536 addresses, /24 = 256 addresses. Smaller number after slash = bigger network.

**Q5. Latency vs bandwidth?**
A: Latency is delay for a single message; bandwidth is how much data can flow per second. Streaming 4K needs bandwidth; a game needs low latency.

**Q6. What is the difference between L4 and L7?**
A: L4 works with IPs/ports (TCP/UDP) and doesn't read content; L7 understands HTTP (URLs, headers, cookies). L7 is smarter, L4 is faster.

---

# 03. DNS Resolution & Traffic Routing

## 📖 Story
You want to call your friend Rahul but don't remember his number. You open your phone's contacts: "Rahul → 98xxxx". **DNS is the phone book of the internet**: you type `amazon.com`, and DNS tells your browser "its IP is `205.251.242.103`".

If your phone doesn't have Rahul, you ask a family member; if they don't know, they ask someone else. That's the **DNS chain**.

## 🧠 Concepts

**Resolution flow (what happens when you type `www.shop.com`)**
1. **Browser cache** → have I looked this up recently?
2. **OS cache** (and `/etc/hosts` file).
3. **Recursive resolver** (your ISP, or 8.8.8.8 Google, 1.1.1.1 Cloudflare). It does the legwork.
4. Resolver asks **Root server** → "who handles `.com`?" → points to **TLD server**.
5. **TLD server (.com)** → "who handles `shop.com`?" → points to **Authoritative name server**.
6. **Authoritative server** → returns the IP (`A` record).
7. Resolver caches it (for the **TTL**) and returns it to you.

**Common DNS record types**
| Record | Meaning |
|---|---|
| A | name → IPv4 |
| AAAA | name → IPv6 |
| CNAME | alias → another name (`www` → `shop.com`) |
| MX | mail servers |
| TXT | free text (verification, SPF) |
| NS | which name servers are authoritative |
| SRV | service location with port |

**TTL (Time To Live)**: how long caches may remember the answer. Low TTL (60s) = changes spread fast but more DNS lookups. High TTL (24h) = fewer lookups but slow to change.

**DNS-based traffic routing (a big interview topic)**
- **Round-robin DNS**: return multiple IPs in rotation (crude load distribution).
- **Weighted routing**: 90% to old version, 10% to new (canary release).
- **Latency-based routing**: send the user to the region with the lowest latency.
- **Geolocation routing**: users in EU → EU servers (also for legal/data residency).
- **Failover routing**: if primary's health check fails, DNS returns the secondary's IP.
- **GeoDNS / Anycast**: many servers share the *same* IP, and the network routes you to the nearest one (used by CDNs and 1.1.1.1).

## 🎯 Analogy
Phone book + a smart receptionist who says, "For Mumbai customers, call the Mumbai branch; if Mumbai is closed, call Pune."

## 🛠 Where it's used
Route 53, Cloudflare DNS, multi-region failover, blue-green deployments, CDN mapping, service discovery in microservices (internal DNS like `payments.internal`).

## ⭐ Important Points
- DNS is a **hierarchical, distributed, cached** system. Caching makes it fast; it also makes changes slow to propagate.
- Lower TTL **before** a planned migration, then raise it afterwards.
- DNS failover is **not instant** because of caching; it can take minutes.
- DNS is a classic **DDoS target** and a potential **SPOF**; use managed, redundant DNS.
- A CNAME can't be at the zone apex (`shop.com`) in standard DNS; providers offer ALIAS/ANAME.

## ❓ Interview Q&A

**Q1. Explain what happens when you type a URL in the browser.**
A: DNS lookup (cache → resolver → root → TLD → authoritative) gives an IP. Browser opens a TCP connection (3-way handshake), does a TLS handshake for HTTPS, sends an HTTP request, gets a response, then renders HTML/CSS/JS and fetches more resources.

**Q2. What is TTL in DNS and what's the trade-off?**
A: It's how long the answer can be cached. Short TTL gives quick changes and failover but more query load/latency; long TTL gives fewer lookups but slow changes.

**Q3. How can DNS help with load balancing?**
A: By returning multiple IPs (round-robin), weighted answers, or location-aware answers. It's coarse-grained, though: it can't see server load in real time and caches delay changes, so real load balancers are still needed.

**Q4. A vs CNAME?**
A: A maps a name directly to an IP. CNAME maps a name to another name, which then resolves to an IP.

**Q5. How does DNS-based failover work?**
A: A health checker monitors the primary; if it fails, the DNS service starts returning the backup IP. Limited by TTL and resolver caching.

**Q6. What is Anycast?**
A: Multiple servers in different places advertise the same IP; routers send each user to the topologically nearest one. Great for CDN, DNS, and DDoS absorption.

---

# 04. TCP, UDP, TLS & Connection Behavior

## 📖 Story
**TCP is like a registered courier**: first you call to confirm the receiver is home (handshake), each parcel is numbered, the receiver confirms each one ("got parcel 3"), and lost ones are re-sent. Slower, but nothing is lost.

**UDP is like shouting across a field**: no confirmation, no resending. If a word is lost, too bad. But it's very fast, perfect for a live cricket commentary where an old missed word is useless anyway.

**TLS is the sealed, tamper-proof envelope** with a lock only the receiver can open, so even the courier can't read your letter.

## 🧠 Concepts

**TCP (Transmission Control Protocol)**
- Connection-oriented, reliable, ordered, with flow control and congestion control.
- **3-way handshake**: `SYN → SYN-ACK → ACK` (costs 1 round trip before data flows).
- Uses sequence numbers, ACKs, retransmissions.
- **Flow control**: don't overwhelm the *receiver* (window size).
- **Congestion control**: don't overwhelm the *network* (slow start, then grow, back off on loss).
- Closing: `FIN`/`ACK` exchange; `TIME_WAIT` state lingers.
- Used by: HTTP/1.1, HTTP/2, databases, SSH, email.

**UDP (User Datagram Protocol)**
- Connectionless, no guarantee of delivery or order, minimal overhead.
- Used by: DNS, video calls, online gaming, live streaming, **QUIC/HTTP/3**.

| | TCP | UDP |
|---|---|---|
| Reliability | Yes | No |
| Ordering | Yes | No |
| Speed | Slower | Faster |
| Header | 20+ bytes | 8 bytes |
| Use | Web, DB, file | Voice, game, DNS |

**TLS (Transport Layer Security) / HTTPS**
Gives **confidentiality** (encryption), **integrity** (not tampered), **authentication** (server is who it claims).
Simplified handshake:
1. Client Hello (supported ciphers, random number).
2. Server Hello + **certificate** (contains server's public key, signed by a trusted CA).
3. Client verifies the certificate chain against trusted CAs.
4. Both derive a shared **session key** (key exchange, e.g. ECDHE).
5. Further traffic is encrypted with the fast **symmetric** key.
(Asymmetric crypto is used only to set up; symmetric does the bulk work, because it's much faster.)
- **TLS 1.3** needs 1 round trip (and supports 0-RTT resumption); TLS 1.2 needs 2.
- **mTLS (mutual TLS)**: both sides present certificates (service-to-service security).

**Connection behavior that matters at scale**
- **Keep-Alive / persistent connections**: reuse a TCP connection for many requests (avoids repeated handshakes).
- **Connection pooling**: app keeps a pool of open DB connections; creating one per request is expensive.
- **HTTP/1.1**: one request at a time per connection (head-of-line blocking) → browsers open ~6 connections.
- **HTTP/2**: multiplexing many streams over one TCP connection, header compression. But TCP-level head-of-line blocking remains.
- **HTTP/3 (QUIC over UDP)**: multiplexing without TCP's head-of-line blocking, faster handshakes, better on mobile networks.
- **Timeouts & retries**: always set connect/read timeouts; retry with **exponential backoff + jitter**.
- **TCP port exhaustion / too many TIME_WAIT**: high-churn short connections can run out of ephemeral ports.

## 🎯 Analogy
TCP = registered post with acknowledgement; UDP = postcards thrown over a fence; TLS = a locked box whose key is exchanged using a secret handshake in a crowded room.

## 🛠 Where it's used
TCP: REST APIs, DB connections. UDP: video conferencing, DNS, games, IoT telemetry. TLS: every HTTPS site, mTLS inside service meshes.

## ⭐ Important Points
- TCP handshake + TLS handshake = latency *before* the first byte. That's why **keep-alive, TLS session resumption, HTTP/2/3, and CDNs near the user** matter.
- Choose UDP when **freshness > completeness** (live data). Choose TCP when **correctness** matters.
- TLS termination is often done at the load balancer/CDN to offload CPU work.
- Retries without backoff can cause a **retry storm** that kills a recovering server.

## ❓ Interview Q&A

**Q1. TCP vs UDP — when to choose which?**
A: TCP when every byte must arrive correctly and in order (payments, web pages, file transfer). UDP when speed and freshness matter more than perfect delivery (video call, gaming, DNS lookups).

**Q2. Explain the TCP 3-way handshake.**
A: Client sends SYN, server replies SYN-ACK, client sends ACK. After this both sides agree on initial sequence numbers and the connection is open.

**Q3. How does TLS work at a high level?**
A: Server proves identity with a CA-signed certificate; client and server agree on a shared secret key using key exchange; after that all data is encrypted with that symmetric key.

**Q4. Why is HTTP/3 faster?**
A: It uses QUIC over UDP, so there's no TCP head-of-line blocking (a lost packet only stalls its own stream), connection setup is faster (handshake combined with TLS), and it handles network switching (Wi-Fi ↔ mobile) well.

**Q5. What is connection pooling and why does it matter?**
A: Reusing a fixed set of established connections (e.g., to a DB) instead of opening a new one per request. Opening connections is slow and DBs have a max connection limit.

**Q6. What's exponential backoff with jitter?**
A: On failure, wait 1s, 2s, 4s, 8s… (exponential), plus a random offset (jitter) so thousands of clients don't retry at the same instant.

**Q7. Flow control vs congestion control?**
A: Flow control protects the receiver from being overwhelmed; congestion control protects the network.

---

# 05. Proxies, CDNs & the Edge

## 📖 Story
**Forward proxy**: At your college hostel, all students go to the internet through the hostel's gate office. The office hides who you are and blocks some sites. *The proxy stands in front of the clients.*

**Reverse proxy**: A big company has a front-desk receptionist. Visitors never meet the employees directly; the receptionist takes the request, forwards it to the right person, and returns the answer. *The proxy stands in front of the servers.*

**CDN**: Imagine a popular book publisher in Delhi. Shipping every copy from Delhi to every city is slow. So they keep **stock in local shops in every city**. When you order, the nearest shop serves you. That's a CDN: copies of your content in many locations near users (**the edge**).

## 🧠 Concepts

**Forward proxy** — sits in front of clients: anonymity, content filtering, corporate access control, caching. (VPN is a cousin.)

**Reverse proxy** — sits in front of servers: load balancing, TLS termination, caching, compression, hiding internal servers, security (WAF). Examples: **Nginx, HAProxy, Envoy, Cloudflare**.

**CDN (Content Delivery Network)**
- Network of **edge servers / PoPs (Points of Presence)** worldwide.
- Caches static content (images, JS, CSS, videos), and increasingly dynamic content/API responses.
- **Origin** = your real server. **Edge** = CDN servers near users.
- **Cache hit** = served from the edge (fast). **Cache miss** = edge fetches from origin, stores, then serves.
- **Pull CDN**: edge fetches from origin on first request (lazy). **Push CDN**: you upload content to the CDN beforehand.
- Controlled by headers: `Cache-Control: max-age=3600, public`, `ETag`, `Last-Modified`.
- **Invalidation/purge**: forcibly remove stale content. Better trick: **versioned file names** (`app.v3.js`, `logo.a1b2c3.png`) so a new version = new URL, no purge needed.
- **Origin shield**: a middle cache layer so many edge misses collapse into one origin request.

**Edge computing**: run code at the edge (Cloudflare Workers, Lambda@Edge): auth checks, A/B tests, redirects, personalization close to the user.

**Extra edge features**: DDoS protection, WAF (web application firewall), bot filtering, TLS termination, image optimization, HTTP/3 support.

## 🎯 Analogy
Reverse proxy = receptionist. Forward proxy = hostel gate. CDN = local branches of a bookshop chain.

## 🛠 Where it's used
Netflix/YouTube videos, e-commerce product images, static websites, software downloads, global API acceleration.

## ⭐ Important Points
- A CDN reduces **latency**, **origin load**, and **bandwidth cost**, and absorbs traffic spikes and DDoS.
- **Cache hit ratio** is the key CDN metric.
- Never cache **private/personalized** data on a shared public cache (e.g., user account pages) unless keyed correctly (`Cache-Control: private`).
- Use **versioned asset URLs** + long TTL for static files.
- Reverse proxy ≠ load balancer, but a reverse proxy often *does* load balancing.

## ❓ Interview Q&A

**Q1. Forward proxy vs reverse proxy?**
A: Forward proxy represents the *client* to the internet (hides client). Reverse proxy represents the *server* to clients (hides servers, balances load, terminates TLS).

**Q2. How does a CDN improve performance?**
A: It serves content from a location geographically near the user, cutting network distance (latency), and it offloads traffic from the origin.

**Q3. How do you update content that's already cached on a CDN?**
A: Either purge/invalidate the URL, or (better) use versioned filenames or short TTLs for changing content; and use `ETag` revalidation.

**Q4. Push vs pull CDN?**
A: Pull: CDN fetches from origin on first miss (easy, most common). Push: you upload content ahead (good for large, rarely-changing files like game patches).

**Q5. What can go wrong with CDN caching?**
A: Stale content, caching personalized responses by mistake, low hit ratio due to varying query strings/cookies, and a *cache stampede* on origin after a purge.

**Q6. What is an origin shield?**
A: A designated CDN layer that sits between edge nodes and your origin so many edge misses become one origin request.

**Q7. Where do you terminate TLS?**
A: Commonly at the CDN/load balancer for performance and certificate management, with re-encryption to the origin for sensitive systems (end-to-end TLS).

---
# 06. Load Balancing & Fault Tolerance

## 📖 Story
Back to the tea stall that became a big food court with 10 counters. If everyone rushes to counter 1, it chokes while others sit idle. You hire a **manager at the entrance**: "You go to counter 4, you go to counter 7, counter 2 is closed today, skip it." That manager is the **load balancer (LB)**.

Now imagine the manager is sick one day. The whole food court stops, even though 10 counters are fine! So you keep a **second manager on standby**. That's **fault tolerance**: the system keeps working even when parts break.

## 🧠 Concepts

**Load Balancer**: distributes incoming traffic across multiple servers to improve throughput, availability, and scalability.

**Types**
- **L4 LB** (transport): routes by IP/port. Very fast, doesn't read HTTP. (AWS NLB)
- **L7 LB** (application): routes by URL path, host, headers, cookies. (AWS ALB, Nginx, Envoy)
  - e.g., `/api/*` → API servers, `/images/*` → image servers.
- **Hardware vs software vs cloud-managed** LBs.
- **Global Server Load Balancing (GSLB)**: across regions (DNS/anycast based).

**Algorithms**
| Algorithm | How | Good when |
|---|---|---|
| Round Robin | 1→2→3→1… | Servers similar, requests similar |
| Weighted Round Robin | Stronger server gets more | Mixed capacity |
| Least Connections | Send to server with fewest active connections | Long/variable requests |
| Least Response Time | Fastest responder wins | Latency-sensitive |
| IP Hash | hash(client IP) → server | Simple stickiness |
| Consistent Hashing | Minimal reshuffle on add/remove | Caches, sharded backends |
| Random (power of two choices) | Pick 2 randomly, choose less loaded | Large-scale, simple, effective |

**Health checks**
- **Active**: LB pings `/health` every few seconds. Unhealthy → removed from pool.
- **Passive**: LB notices failures/timeouts in real traffic.
- Good health endpoints check *real readiness* (DB reachable?) but not too deeply, or a DB blip removes all servers at once.

**Sticky sessions (session affinity)**: same user → same server (via cookie). Simple but harms balance and breaks on server death. Prefer **stateless servers + shared session store**.

**Fault tolerance & high availability techniques**
- **Redundancy**: N+1 or N+2 servers; no SPOF.
- **Active-Active**: all nodes serve traffic. **Active-Passive**: standby takes over on failure (failover).
- **Multi-AZ / Multi-region** deployment.
- **Graceful degradation**: if recommendations fail, still show products.
- **Circuit breaker**: stop calling a failing service for a while (explained more in #21).
- **Timeouts, retries (with backoff), bulkheads** (isolate resources so one failure doesn't drown everything).
- **Connection draining**: when removing a server, let in-flight requests finish.
- **Who balances the balancer?** Run LBs in pairs/clusters using virtual IP (keepalived/VRRP), anycast, or managed cloud LBs.

**Availability numbers**: 99.9% ("three nines") ≈ 8.76 hours downtime/year; 99.99% ≈ 52 minutes/year; 99.999% ≈ 5 minutes/year.

## 🎯 Analogy
Supermarket billing: a floor manager directs customers to the shortest queue, closes a broken billing machine, and keeps a spare one ready.

## 🛠 Where it's used
Every production web service: AWS ALB/NLB, Nginx, HAProxy, Envoy, Kubernetes Services/Ingress, Google Cloud LB.

## ⭐ Important Points
- LB itself must be redundant (avoid SPOF).
- **Health checks + auto-removal** is what turns an LB into a fault-tolerance tool.
- L7 gives smart routing; L4 gives raw speed.
- Stateless servers make *any* algorithm work easily.
- Test failures on purpose (**chaos engineering**).

## ❓ Interview Q&A

**Q1. How does a load balancer improve availability?**
A: It detects unhealthy servers through health checks and stops sending traffic to them, so users never notice a single server dying.

**Q2. L4 vs L7 load balancing?**
A: L4 forwards TCP/UDP packets based on IP/port (fast, simple). L7 understands HTTP, so it can route by path, header, cookie, and do TLS termination, rate limiting, and rewrites.

**Q3. Round robin vs least connections?**
A: Round robin is simple and equal-share; fine when requests cost about the same. Least connections adapts when some requests are long-running, avoiding piling up on a slow server.

**Q4. What are sticky sessions and why avoid them?**
A: They pin a user to one server. They can unbalance load and lose sessions on server failure. Better to store sessions in Redis or use JWT.

**Q5. How do you make the load balancer not a SPOF?**
A: Run multiple LB instances across AZs with a floating IP/anycast/DNS, or use managed cloud LBs that are inherently redundant.

**Q6. Active-active vs active-passive?**
A: Active-active uses all capacity and fails over smoothly but needs data consistency handling. Active-passive is simpler, but standby capacity sits idle and failover takes time.

**Q7. What is graceful degradation?**
A: When a non-critical dependency fails, return reduced functionality instead of an error. Example: show a default "popular items" list if the recommendation service is down.

---

# 07. RESTful APIs & HTTP Contracts

## 📖 Story
A restaurant has a **menu** (the API contract). You, the customer, don't enter the kitchen; you say "I'd like item #5" in a standard way, and the kitchen replies in a standard way. Anyone who knows the menu can order, whether they're from India or Italy. **REST is a set of rules for designing such a menu for the web**, using HTTP as the language.

## 🧠 Concepts

**REST (Representational State Transfer)** — architecture style where everything is a **resource** identified by a **URL**, manipulated using standard **HTTP methods**.

**Resources & URLs (use nouns, not verbs)**
```
GET    /products          → list products
GET    /products/42       → get product 42
POST   /products          → create product
PUT    /products/42       → replace product 42
PATCH  /products/42       → partially update
DELETE /products/42       → delete
GET    /users/7/orders    → orders of user 7
```
❌ Bad: `/getProduct`, `/createOrder`. ✅ Good: `GET /products/42`, `POST /orders`.

**HTTP methods: safe & idempotent**
| Method | Safe (no change) | Idempotent (same result if repeated) |
|---|---|---|
| GET | ✅ | ✅ |
| HEAD / OPTIONS | ✅ | ✅ |
| PUT | ❌ | ✅ |
| DELETE | ❌ | ✅ |
| PATCH | ❌ | Not guaranteed |
| POST | ❌ | ❌ |

**Status codes**
- **2xx success**: 200 OK, 201 Created, 202 Accepted (async), 204 No Content.
- **3xx redirect**: 301 permanent, 302 temporary, 304 Not Modified (cache valid).
- **4xx client error**: 400 Bad Request, 401 Unauthorized (not authenticated), 403 Forbidden (no permission), 404 Not Found, 405 Method Not Allowed, 409 Conflict, 422 Unprocessable, 429 Too Many Requests.
- **5xx server error**: 500 Internal, 502 Bad Gateway, 503 Service Unavailable, 504 Gateway Timeout.

**Important headers**: `Content-Type`, `Accept`, `Authorization`, `Cache-Control`, `ETag`, `If-None-Match`, `Idempotency-Key`, `Location`, `Retry-After`.

**REST principles**
1. Client-server separation.
2. **Stateless** (each request has everything needed).
3. **Cacheable** responses.
4. Uniform interface.
5. Layered system (proxies, gateways in between).
6. (Optional) Code on demand.
(HATEOAS = responses include links to next actions; rarely fully used.)

**Good API design practices**
- **Versioning**: `/v1/products` or header-based. Never break existing clients.
- **Pagination**: offset (`?page=2&limit=20`) is simple but slow/inconsistent at scale; **cursor/keyset** (`?after=abc123`) is stable and fast.
- **Filtering/sorting**: `?category=shoes&sort=-price`.
- **Consistent error format**: `{ "code": "OUT_OF_STOCK", "message": "...", "details": [...] }`.
- **Idempotency keys** for POST payments/orders so retries don't double-charge.
- **Partial responses / field selection**, **ETags** for caching & optimistic concurrency.
- **Bulk operations** for efficiency; **async operations** return `202 Accepted` + a status URL.
- **Backward compatibility**: add fields, don't remove/rename.

**PUT vs PATCH**: PUT sends the whole object (replace). PATCH sends only changes.

## 🎯 Analogy
Menu card = API spec. Dish names = resources. "Order / modify / cancel / view" = POST / PUT or PATCH / DELETE / GET. Bill number to avoid double billing = idempotency key.

## 🛠 Where it's used
Almost every web/mobile backend, public APIs (Stripe, GitHub, Twitter), internal microservice endpoints.

## ⭐ Important Points
- Use **nouns, plural, hierarchical** URLs.
- Right method + right status code = self-explanatory API.
- GET must never change data (crawlers & prefetchers call it!).
- **401 vs 403**: 401 = "who are you?", 403 = "I know you, but no."
- Design for **retries**: network fails, clients retry, so make writes idempotent.

## ❓ Interview Q&A

**Q1. What makes an API RESTful?**
A: Resource-based URLs, correct use of HTTP verbs and status codes, statelessness, cacheability, and a uniform interface.

**Q2. PUT vs POST vs PATCH?**
A: POST creates (server often decides ID). PUT replaces a resource at a known URL (idempotent). PATCH partially updates.

**Q3. What does idempotent mean and why does it matter?**
A: Repeating the same request gives the same end state. It matters because networks fail and clients retry; without idempotency a retried "pay" could charge twice. For POST I use an `Idempotency-Key` header and store the result by that key.

**Q4. Offset vs cursor pagination?**
A: Offset (`LIMIT 20 OFFSET 10000`) gets slow because the DB scans/skips rows, and results shift if data changes. Cursor pagination uses "give me items after this ID/timestamp", which is fast with an index and stable.

**Q5. 401 vs 403?**
A: 401 = not authenticated (missing/invalid credentials). 403 = authenticated but not allowed.

**Q6. How do you version an API?**
A: URL (`/v2/…`), header, or media type. Keep old versions alive for a deprecation period; prefer additive, backward-compatible changes.

**Q7. How would you handle a long-running operation via REST?**
A: Return `202 Accepted` with a `Location` header pointing to a status resource (`/jobs/123`). Client polls or receives a webhook.

**Q8. What is HTTP caching with ETag?**
A: Server sends `ETag: "v5"`. Client later sends `If-None-Match: "v5"`. If unchanged, server returns `304 Not Modified` with no body, saving bandwidth.

---

# 08. API Gateway, Authentication & Rate Limiting

## 📖 Story
A big office building has **one main gate**. The security guard checks your ID card (**authentication**), checks whether you're allowed on the 5th floor (**authorization**), counts how many times you've entered today (**rate limiting**), and directs you to the right department (**routing**). Employees inside don't need to verify every visitor again.

That guard is the **API Gateway**: the single front door for all your microservices.

## 🧠 Concepts

**API Gateway** — single entry point that handles cross-cutting concerns:
- Routing requests to the right microservice.
- Authentication & authorization.
- Rate limiting / throttling / quotas.
- TLS termination, request/response transformation.
- Caching, logging, metrics, tracing.
- Protocol translation (REST ↔ gRPC), request aggregation.
- Examples: Kong, AWS API Gateway, Apigee, NGINX, Envoy.
- **BFF (Backend For Frontend)**: separate gateway per client type (web, mobile) tailoring responses.

**Authentication vs Authorization**
- **AuthN** = *who are you?* (login, password, OTP, biometrics).
- **AuthZ** = *what can you do?* (roles, permissions).

**Common auth mechanisms**
| Method | Idea |
|---|---|
| Session + Cookie | Server stores session; browser holds session ID cookie. Stateful. |
| API Key | Static key per client; simple, identifies app (not great as sole security). |
| JWT (JSON Web Token) | Signed token containing claims (user id, roles, expiry). Server verifies the signature without a DB lookup. Stateless. |
| OAuth 2.0 | **Delegated authorization**: "Let this app access my Google Calendar without giving it my password." Issues access tokens. |
| OpenID Connect (OIDC) | Authentication layer on top of OAuth2 ("Login with Google"). |
| mTLS | Both sides show certificates (service-to-service). |

**JWT structure**: `header.payload.signature` (Base64). Signature proves it wasn't altered. **Short-lived access token** (5–15 min) + **refresh token** (long-lived, stored securely) is the standard.
JWT downside: hard to revoke before expiry → keep short life, or maintain a denylist.

**OAuth2 flow (Authorization Code, simplified)**
1. App redirects the user to Google's login.
2. User logs in and consents.
3. Google returns an **authorization code** to the app.
4. App exchanges code (+ secret/PKCE) for an **access token**.
5. App calls API with `Authorization: Bearer <token>`.

**Rate limiting** — control how many requests a client can make, to protect from abuse, DDoS, noisy neighbors, and to ensure fair use and cost control.

Algorithms:
- **Fixed window**: max 100 req per minute-window. Simple; allows bursts at window boundaries (100 at 12:00:59 + 100 at 12:01:00 = 200 in 2 seconds).
- **Sliding window log/counter**: smoother and more accurate; more memory.
- **Token bucket** ⭐: bucket holds N tokens, refills at a constant rate; each request takes a token. Allows **bursts** up to bucket size while enforcing average rate. Most popular.
- **Leaky bucket**: requests drain at a constant rate (smooths traffic into steady output).

**Distributed rate limiting**: keep counters in **Redis** (atomic `INCR` + `EXPIRE`, or Lua scripts) so all gateway nodes share the same view. Key by user ID, API key, or IP.
Response: `429 Too Many Requests` with `Retry-After` and headers like `X-RateLimit-Remaining`.

**Related protections**: throttling (slow down instead of reject), quotas (monthly caps), load shedding (drop low-priority traffic when overloaded), WAF, bot detection.

## 🎯 Analogy
Security guard + reception + turnstile that lets only X people per minute. Token bucket = a coin dispenser that adds one coin per second up to a max; each entry costs one coin.

## 🛠 Where it's used
Public APIs (Stripe-like quotas), login endpoints (brute-force protection), microservice platforms, SaaS pricing tiers.

## ⭐ Important Points
- Centralizing auth/rate limiting in the gateway avoids repeating it in every service.
- Gateway is critical → must be **highly available and scalable** (it can become a SPOF/bottleneck).
- Rate limit by **multiple dimensions** (per user, per IP, per endpoint).
- Don't put heavy business logic in the gateway.
- Never store secrets/tokens in `localStorage` if you can avoid it (XSS risk); prefer `HttpOnly`, `Secure` cookies.
- Fail-open vs fail-closed: if the rate-limit store (Redis) is down, decide whether to allow all or block all (usually fail-open for availability, fail-closed for security-critical endpoints).

## ❓ Interview Q&A

**Q1. What is an API Gateway and why use one?**
A: A single entry point for clients that handles routing, auth, rate limiting, logging, and more, so individual microservices stay simple and clients don't need to know the internal topology.

**Q2. AuthN vs AuthZ?**
A: Authentication proves identity; authorization decides permissions after identity is known.

**Q3. Session vs JWT?**
A: Session: state on the server, easy revocation, needs shared store when scaled. JWT: state inside a signed token, no lookup needed so scales easily, but harder to revoke and bigger in size.

**Q4. Explain OAuth 2.0 in simple words.**
A: It lets a user grant a third-party app limited access to their data on another service, without sharing their password. The app receives an access token with specific scopes.

**Q5. Design a rate limiter.**
A: Use token bucket per user/API key. Store `tokens` and `last_refill` in Redis, updated atomically via a Lua script. Gateway checks before forwarding; return 429 + `Retry-After` when empty. Use sliding/token bucket to avoid boundary bursts, config rules per endpoint/tier, and make it a separate, horizontally-scaled component.

**Q6. Fixed window vs token bucket?**
A: Fixed window is simple but has boundary-burst problems. Token bucket allows controlled bursts while maintaining a long-run average rate, smoother and more flexible.

**Q7. How do you secure service-to-service calls?**
A: mTLS (often via service mesh), or signed short-lived tokens; network policies; least-privilege permissions.

**Q8. How do you revoke a JWT?**
A: Use short expiry + refresh tokens; keep a denylist (by token ID) in a fast store; rotate signing keys in emergencies.

---

# 09. CORS & Browser Security

## 📖 Story
You live in a gated society. A stranger calls the guard saying "I'm from Flat 12, let me into Flat 7." The guard says: "Rules: a person from outside our society can enter only if Flat 7's owner has told me in advance that this person is allowed." 

Browsers behave like that guard. A script from `evil.com` shouldn't be able to silently read your data from `bank.com` using your logged-in cookies. This protection is called the **Same-Origin Policy**. **CORS** is the owner's written permission slip that says "yes, `app.mysite.com` is allowed to read my responses."

## 🧠 Concepts

**Origin** = scheme + host + port. `https://app.shop.com:443`. Different in any part → different origin.
| URL A | URL B | Same origin? |
|---|---|---|
| https://shop.com | https://shop.com/cart | ✅ |
| https://shop.com | http://shop.com | ❌ (scheme) |
| https://shop.com | https://api.shop.com | ❌ (host) |
| https://shop.com | https://shop.com:8443 | ❌ (port) |

**Same-Origin Policy (SOP)**: browser blocks JS on one origin from *reading* responses from another origin.

**CORS (Cross-Origin Resource Sharing)**: server uses HTTP headers to tell the browser which other origins may read its responses.
- `Access-Control-Allow-Origin: https://app.shop.com` (or `*`)
- `Access-Control-Allow-Methods: GET, POST, PUT`
- `Access-Control-Allow-Headers: Content-Type, Authorization`
- `Access-Control-Allow-Credentials: true` (allow cookies; then origin **cannot** be `*`)
- `Access-Control-Max-Age: 600` (cache the preflight)

**Simple vs preflighted requests**
- **Simple** (GET/POST with basic headers/content types): sent directly; browser checks the response headers.
- **Preflight**: for "non-simple" requests (PUT/DELETE, custom headers like `Authorization`, `application/json`), the browser first sends an **`OPTIONS`** request asking "may I?" The server answers with allowed methods/headers; only then the real request is sent.

**Important truths about CORS**
- CORS is enforced **by the browser only**. curl, Postman, and server-to-server calls ignore it.
- CORS is **not a security measure against attackers**; it *relaxes* SOP for trusted origins. It protects users' browsers.
- A CORS error means the *server* didn't allow your origin; the fix is on the server (or via a same-origin proxy), not in front-end code.

**Other browser security topics**
- **XSS (Cross-Site Scripting)**: attacker injects JS into your page. Defenses: escape/sanitize output, **CSP (Content-Security-Policy)**, `HttpOnly` cookies.
- **CSRF (Cross-Site Request Forgery)**: evil site makes your browser send an authenticated request to your bank. Defenses: **SameSite cookies**, CSRF tokens, checking `Origin`/`Referer`.
- **Cookie flags**: `HttpOnly` (JS can't read), `Secure` (HTTPS only), `SameSite=Lax/Strict`.
- **HSTS**: force HTTPS. **Clickjacking** defense: `X-Frame-Options` / CSP `frame-ancestors`.
- **Never** use `Access-Control-Allow-Origin: *` together with sensitive credentialed APIs; **never reflect** any origin blindly.

## 🎯 Analogy
Society guard checks a pre-approved visitor list (CORS allow-list). The preflight is the guard phoning the owner first before letting a risky visitor in.

## 🛠 Where it's used
Any SPA (React/Angular) at `app.example.com` calling an API at `api.example.com`; third-party widgets; public APIs consumed from browsers.

## ⭐ Important Points
- Preflight adds an extra round trip → set `Access-Control-Max-Age` to cache it.
- Allow-list exact origins; avoid wildcard for credentialed requests.
- CORS ≠ authentication. You still need proper auth on the API.
- CORS is configured on the **API gateway/server**, not in the browser.

## ❓ Interview Q&A

**Q1. What is CORS and why does it exist?**
A: Browsers enforce Same-Origin Policy so a malicious site can't read data from another site using your logged-in session. CORS lets a server explicitly allow specific other origins to read its responses.

**Q2. What is a preflight request?**
A: An automatic `OPTIONS` request the browser sends before a "non-simple" cross-origin request, to ask the server whether the actual method/headers are permitted.

**Q3. Why does Postman work but my browser gives a CORS error?**
A: CORS is a browser-enforced rule. Postman/curl don't enforce it. Fix by configuring the server's CORS headers.

**Q4. Is `Access-Control-Allow-Origin: *` safe?**
A: Fine for public, non-credentialed, public data. Unsafe for authenticated APIs; and with `Allow-Credentials: true` the wildcard isn't allowed anyway.

**Q5. How do CSRF and CORS differ?**
A: CSRF tricks a browser into *sending* an authenticated request (the attacker doesn't need to read the response). CORS governs whether a page can *read* cross-origin responses. SameSite cookies and CSRF tokens defend against CSRF.

**Q6. How do you prevent XSS?**
A: Output-encode user data, sanitize HTML, use CSP, avoid `innerHTML`, use `HttpOnly` cookies so scripts can't steal sessions.

**Q7. Can you avoid CORS entirely?**
A: Yes, by serving front-end and API from the same origin (e.g., reverse proxy routes `/api` to the backend).

---

# 10. REST, GraphQL, gRPC & Real-Time Communication

## 📖 Story
Four ways to get food:
- **REST** = a fixed **set menu**. You order "meal #3" and get everything in it, even the pickles you don't like (over-fetching), or you need two orders to get both a main and a drink (under-fetching).
- **GraphQL** = **build-your-own bowl**. "Give me rice, paneer, no onions." You get exactly what you asked for in one order.
- **gRPC** = a **dedicated hotline between two kitchens**, using a compact shorthand language. Super fast, used behind the scenes.
- **Real-time (WebSocket / SSE / polling)** = the **waiter keeps standing at your table** and tells you the moment each dish is ready, instead of you asking every minute.

## 🧠 Concepts

### REST (recap)
Resource URLs + HTTP verbs + JSON. Simple, cacheable, universal. Weakness: over-fetching, under-fetching, many round trips, versioning pain.

### GraphQL
- **Single endpoint** (`POST /graphql`); client sends a **query** describing exactly the fields needed.
- Strongly typed **schema**; supports **queries** (read), **mutations** (write), **subscriptions** (real-time).
```graphql
query {
  product(id: 42) { name price reviews(limit: 3) { rating text } }
}
```
- ✅ No over/under-fetching, great for mobile & complex UIs, one round trip, self-documenting.
- ❌ HTTP caching is harder (all POST to one URL), **N+1 query problem** (fix with DataLoader/batching), complex rate limiting (cost-based), query depth abuse (limit depth/complexity), file upload and error-handling are awkward.

### gRPC
- Google's RPC framework: **Protocol Buffers (binary)** + **HTTP/2** (multiplexing, streaming).
- Define service in `.proto`; code is auto-generated in many languages.
- Four modes: unary, server streaming, client streaming, bidirectional streaming.
- ✅ Very fast, small payloads, strong contracts, streaming, ideal for **internal microservices**.
- ❌ Not human-readable, limited native browser support (needs gRPC-Web), harder to debug with curl, less cache-friendly.

### Comparison
| | REST | GraphQL | gRPC |
|---|---|---|---|
| Format | JSON/HTTP | JSON, single endpoint | Protobuf (binary), HTTP/2 |
| Speed | Good | Good | Excellent |
| Browser-friendly | ✅ | ✅ | ⚠️ (needs proxy) |
| Caching | Easy (HTTP) | Harder | Hard |
| Best for | Public APIs, CRUD | Flexible UIs, mobile, aggregation | Internal service-to-service, low latency |
| Streaming | Limited | Subscriptions | Native |

### Real-time communication options
1. **Short polling**: client asks "any news?" every N seconds. Simple, wasteful, delayed.
2. **Long polling**: client asks; server *holds* the request open until there's news (or timeout), then client re-asks. Better than short polling; still many requests.
3. **Server-Sent Events (SSE)**: one-way stream **server → client** over HTTP, auto-reconnect. Great for notifications, live scores, dashboards.
4. **WebSocket**: persistent **two-way (full-duplex)** connection over a single TCP connection after an HTTP upgrade. Chat, multiplayer games, collaborative editing, trading.
5. **WebRTC**: peer-to-peer audio/video/data (UDP-based), used in video calls.
6. **Webhooks**: server-to-server callbacks ("when payment is done, POST to my URL").

**Scaling WebSockets**: connections are long-lived and stateful → need LB with connection-aware routing, a **pub/sub backbone (Redis Pub/Sub, Kafka)** so a message received by server A reaches the user connected to server B, heartbeats/ping-pong, reconnect with backoff, and handling millions of open sockets (OS file-descriptor limits, memory).

## 🎯 Analogy
REST = set menu, GraphQL = custom bowl, gRPC = kitchen-to-kitchen walkie-talkie, WebSocket = a phone call that stays connected, SSE = a radio broadcast, polling = repeatedly knocking on the door.

## 🛠 Where it's used
REST: public APIs. GraphQL: GitHub API, Shopify, mobile apps. gRPC: Google, Netflix, Uber internal calls. WebSocket: WhatsApp Web, Slack, Zerodha live prices. SSE: live notifications, ChatGPT-style streaming responses.

## ⭐ Important Points
- A common pattern: **external REST/GraphQL, internal gRPC**.
- GraphQL shifts complexity from the client to the server (caching, security, N+1).
- WebSocket needs **sticky routing or pub/sub** to scale.
- If you only need server→client updates, **SSE is simpler** than WebSocket.
- Use **webhooks** for server-to-server event notification instead of polling.

## ❓ Interview Q&A

**Q1. REST vs GraphQL — when to choose which?**
A: REST for simple resource-oriented, cache-friendly public APIs. GraphQL when clients (especially mobile) need flexible, aggregated data from many sources in one call and over/under-fetching hurts.

**Q2. Why is gRPC fast?**
A: Binary Protobuf is compact and quick to serialize, and HTTP/2 gives multiplexing and header compression over a persistent connection.

**Q3. What's the N+1 problem in GraphQL?**
A: Fetching a list of N items then making one extra DB call per item for nested data (1 + N queries). Solve with batching/caching (DataLoader) or joins.

**Q4. WebSocket vs SSE vs long polling?**
A: WebSocket = two-way, low latency, best for chat/games. SSE = one-way server push, simple, HTTP-friendly. Long polling = fallback, works everywhere but heavier.

**Q5. How would you scale a chat system using WebSockets?**
A: Many WS servers behind an LB; a connection registry (user → server) in Redis; messages routed via pub/sub or Kafka so any server can deliver to the right socket; heartbeats; persist messages in a DB for offline users; push notifications for disconnected devices.

**Q6. How do you secure GraphQL?**
A: Auth on resolvers, query depth/complexity limits, persisted queries, disable introspection in production if needed, rate limit by cost.

**Q7. Why not use gRPC directly from browsers?**
A: Browsers can't fully control HTTP/2 framing needed by gRPC; you need gRPC-Web with a proxy (like Envoy) to translate.

**Q8. What is a webhook and how do you make it reliable?**
A: An HTTP callback sent by a service when an event occurs. Make it reliable with retries + backoff, signatures (HMAC) for authenticity, idempotent handlers, and a dead-letter queue.

---
# 11. Caching Patterns & Invalidation

## 📖 Story
You're a student who often needs the same formula from a thick textbook in the library (the **database**). Walking to the library each time is slow. So you write the formula on a **sticky note on your desk** (the **cache**). Next time: instant!

But the sticky note has problems: space is small (what do you throw away?), and if the textbook is updated, your note is now **wrong** (stale). As the famous line goes: *"There are only two hard things in computer science: cache invalidation and naming things."*

## 🧠 Concepts

**Cache** = fast, small, temporary storage in front of a slower, bigger source. Works because of **locality** (same data is asked again and again; the 80/20 rule).

**Where caching happens (layers)**
1. **Browser cache** (HTTP caching headers)
2. **CDN / edge cache**
3. **API gateway / reverse proxy cache**
4. **Application cache** (in-process memory like Caffeine/Guava; or **distributed cache** like Redis/Memcached)
5. **Database cache** (buffer pool, query cache)
6. **CPU caches** (hardware)

**Key metrics**: **hit ratio** = hits / (hits + misses). **Latency** of hit vs miss.

**Caching patterns (very important)**
1. **Cache-Aside (Lazy Loading)** ⭐ most common
   - App checks cache → if miss, reads DB → stores in cache → returns.
   - Writes go to DB, then **invalidate/delete** the cache key.
   - ✅ Only caches what's used, resilient to cache failure. ❌ First request is slow; stale data possible.
2. **Read-Through**: cache itself loads from DB on a miss (app only talks to the cache).
3. **Write-Through**: write goes to cache **and** DB synchronously. ✅ Cache always fresh. ❌ Slower writes; caches data that may never be read.
4. **Write-Back (Write-Behind)**: write to cache, flush to DB later asynchronously. ✅ Very fast writes. ❌ Risk of data loss if cache dies before flush.
5. **Write-Around**: write directly to DB, skip cache; cache fills on reads. Good when written data is rarely read soon.
6. **Refresh-Ahead**: proactively refresh popular keys before they expire.

**Eviction policies** (when cache is full)
- **LRU** (Least Recently Used) ⭐ — most common.
- **LFU** (Least Frequently Used), **FIFO**, **Random**, **TTL-based** expiry.

**Invalidation strategies**
- **TTL (time-based)**: data expires after N seconds. Simple, tolerates slight staleness.
- **Explicit invalidation**: delete/update the key when data changes (on write, or via events/CDC).
- **Versioned keys**: `product:42:v7`. New version → new key.
- **Event-driven invalidation**: DB change → message on a queue → cache updated/deleted.

**Classic caching problems**
| Problem | What happens | Fix |
|---|---|---|
| **Cache stampede / thundering herd** | A hot key expires; thousands of requests hit the DB at once | Request coalescing/locking (single-flight), TTL jitter, refresh-ahead, serve-stale-while-revalidate |
| **Cache penetration** | Requests for keys that **don't exist** always miss and hit DB | Cache null results (short TTL), **Bloom filter**, input validation |
| **Cache avalanche** | Many keys expire at the same time (or cache cluster dies) | Randomize TTLs, multi-layer cache, replicas, circuit breakers |
| **Hot key** | One key gets extreme traffic and overloads a single cache node | Local in-process cache, key replication/splitting |
| **Stale data / inconsistency** | Cache differs from DB | TTL, invalidate-on-write, accept eventual consistency |
| **Race conditions** | Concurrent read/write leave old value in cache | Delete (not update) on write, short TTL, versioning |

**Redis vs Memcached**: Redis has rich data structures (lists, sets, sorted sets, hashes), persistence, replication, pub/sub, clustering, Lua scripts. Memcached is simple, multi-threaded, purely key-value.

**Distributed cache scaling**: shard keys across nodes using **consistent hashing**; replicate for HA.

**HTTP caching headers**: `Cache-Control: max-age, no-cache (revalidate), no-store (never cache), private, public, stale-while-revalidate`, `ETag`, `Last-Modified`.

## 🎯 Analogy
Sticky note on your desk vs textbook in the library. TTL = "throw this note away after 1 hour". Stampede = the whole class rushing to the library when the note on the class board is erased at the same moment.

## 🛠 Where it's used
Product pages, user sessions, leaderboards (Redis sorted sets), rate limiters, feed/timeline, config data, expensive query results, API responses.

## ⭐ Important Points
- Cache **read-heavy, rarely-changing** data first.
- **Cache-aside + TTL + delete-on-write** is the default answer.
- Prefer **deleting** the key on write over updating it (avoids race conditions).
- **Never treat the cache as the source of truth** (unless designed for it).
- Always consider: *what if cache is down?* (system should still work, just slower).
- Add **jitter** to TTLs.

## ❓ Interview Q&A

**Q1. Explain cache-aside.**
A: App checks the cache; on a miss it reads the DB, puts the result in the cache, and returns it. On updates, app writes to DB and invalidates the cache key. The app controls everything, and a cache failure doesn't break correctness.

**Q2. Write-through vs write-back?**
A: Write-through writes to cache and DB together (consistent but slower). Write-back writes to the cache and persists later (fast but risks losing data if the cache crashes).

**Q3. What's a cache stampede and how do you prevent it?**
A: When a popular key expires, many requests simultaneously miss and hammer the DB. Prevent with locks/single-flight so one request rebuilds the value, staggered/jittered TTLs, refresh-ahead, or serving stale data while refreshing.

**Q4. Cache penetration vs avalanche?**
A: Penetration = queries for non-existent data bypass the cache every time (fix: cache nulls, Bloom filter). Avalanche = many keys expire at once or cache goes down, flooding the DB (fix: random TTL, replicas, rate limits).

**Q5. How do you keep cache and DB consistent?**
A: Perfect consistency is costly. Common approach: write to DB, then delete cache key; use short TTL as safety net; for stricter needs use CDC (change data capture) events to invalidate/update, or versioned keys.

**Q6. Which eviction policy would you choose?**
A: LRU by default since recent items are likely to be reused; LFU if popularity is stable and you want frequently-used items to survive scans.

**Q7. Why delete the cache instead of updating it on writes?**
A: Two concurrent writers can update the cache out of order, leaving stale data. Deleting forces the next read to fetch the latest from the DB, which is simpler and safer.

**Q8. Where would you put caches for an e-commerce site?**
A: CDN for images/static assets, Redis for product details/session/cart, local in-memory cache for tiny config/hot data, HTTP caching headers for API responses, DB buffer pool automatically.

---

# 12. Scalability: Horizontal, Vertical & Diagonal Scaling

## 📖 Story
Your small pizza shop gets popular. Options:
1. **Vertical scaling (scale up)**: buy a *bigger oven*, hire a *super-chef*. Simple, but there's a limit to how big an oven can be, it's costly, and if it breaks, you're closed.
2. **Horizontal scaling (scale out)**: open *more shops* in different areas with identical menus. No hard limit, and if one shop closes the others continue. But now you must manage deliveries, keep recipes in sync, and route customers.
3. **Diagonal scaling**: first upgrade your oven until it gets too expensive, *then* open more shops, each with a good-sized oven. Practical real-world mix.

## 🧠 Concepts

**Scalability** = ability to handle growing load by adding resources without redesign or big performance degradation.

| | Vertical (Up) | Horizontal (Out) | Diagonal |
|---|---|---|---|
| What | Bigger machine (more CPU/RAM/disk) | More machines | Scale up to a sweet spot, then out |
| Complexity | Low | High (distribution) | Medium |
| Limit | Hardware ceiling | Practically unlimited | Flexible |
| Downtime | Often needs restart | Usually none | Mixed |
| Fault tolerance | Single point of failure | Redundancy built-in | Good |
| Cost curve | Exponential at high end | Linear-ish (commodity) | Balanced |

**What makes horizontal scaling possible**
- **Stateless** app tiers (sessions externalized).
- **Load balancer** in front.
- **Shared nothing** architecture.
- **Data tier scaling**: replication (read scaling), sharding/partitioning (write + storage scaling), caching, CQRS.
- **Async processing** via queues.
- **Service decomposition** (microservices).

**Scaling each layer**
| Layer | Techniques |
|---|---|
| Web/App | Stateless + LB + autoscaling |
| Database reads | Read replicas, caching, indexes |
| Database writes | Sharding, partitioning, queues, batching |
| Static content | CDN, object storage |
| Heavy tasks | Queues + worker pools |
| Search/analytics | Dedicated engines (Elasticsearch, OLAP) |

**Other related terms**
- **Throughput** (requests/sec), **capacity planning**, **bottleneck** (the slowest part limits the whole system).
- **Amdahl's Law**: the part that can't be parallelized limits the speedup.
- **Read-heavy vs write-heavy**: read-heavy → caching + replicas; write-heavy → sharding + queues + append-only/LSM stores.
- **Hot spots**: uneven load on one shard/partition.
- **Elasticity**: scaling *both up and down* automatically (cloud) vs scalability (ability to grow).

**Scaling stages (typical journey)**
1. Single server (app + DB).
2. Separate DB from app server.
3. Add LB + multiple app servers (stateless).
4. Add cache + CDN.
5. DB read replicas.
6. Queues + async workers.
7. Shard the database / microservices.
8. Multi-region + global traffic management.

## 🎯 Analogy
Vertical = bigger truck. Horizontal = more trucks. Diagonal = upgrade the truck until it's economical, then buy a fleet.

## 🛠 Where it's used
Every growing startup; Netflix/Amazon-scale systems are horizontally scaled; legacy relational DB servers are often scaled vertically first.

## ⭐ Important Points
- Start simple; **vertical first is a perfectly valid early choice** (cheap & quick), but plan for horizontal.
- Horizontal scaling needs **stateless services** and **partitioned data**.
- Databases are the hardest layer to scale horizontally.
- Find the **bottleneck** before scaling (measure, don't guess).
- Scaling out increases failure modes → need monitoring, retries, idempotency.

## ❓ Interview Q&A

**Q1. Vertical vs horizontal scaling — pros and cons?**
A: Vertical is simple (no code changes) but capped by hardware, costly, and a SPOF. Horizontal scales almost without limit and improves availability but adds complexity like data distribution, consistency, and load balancing.

**Q2. What is diagonal scaling?**
A: A hybrid: scale up each node to a cost-efficient size, then scale out by adding more such nodes as load grows.

**Q3. How do you make an application horizontally scalable?**
A: Make servers stateless, externalize sessions/files to Redis/object storage, put an LB in front, use shared/partitioned data stores, and move heavy work to async queues.

**Q4. How would you scale a read-heavy database?**
A: Add indexes, caching, then read replicas (route reads to replicas, writes to primary). Mind replication lag.

**Q5. How would you scale a write-heavy database?**
A: Shard by a good key, batch writes, use queues to smooth spikes, consider write-optimized stores (Cassandra/LSM-tree), and reduce unnecessary indexes.

**Q6. Scalability vs elasticity?**
A: Scalability = can the system grow to handle more load. Elasticity = it grows *and shrinks* automatically with demand.

**Q7. How do you identify a bottleneck?**
A: Load test, monitor CPU/memory/IO/network, look at DB slow query logs, trace requests, and check saturation metrics (queue depth, connection pool wait). Fix the biggest constraint, then re-measure.

---

# 13. Autoscaling & Cloud Operational Practices

## 📖 Story
A cinema hall hires extra ticket counters on Friday evenings and sends staff home on Tuesday afternoons. It does this using a **rule**: "If the queue exceeds 50 people, open 2 more counters; if under 10 for 15 minutes, close one." That's **autoscaling**: your cloud capacity follows demand automatically; you pay only for what you use and don't crash at peak.

## 🧠 Concepts

**Autoscaling** — automatically adjusting the number (or size) of compute resources based on load.

**Types**
- **Reactive / Dynamic**: scale when metrics cross thresholds (CPU > 70%, requests per instance, queue length, latency).
- **Scheduled**: scale at known times (sale at 8 PM, month-end billing).
- **Predictive**: ML forecasts demand and scales ahead of time.
- **Target tracking**: "keep average CPU at 50%"; system calculates the needed instances.
- **Step scaling**: different responses for different breach levels.

**Key controls**
- **Min / Max / Desired** instance counts (min for baseline availability, max for cost protection).
- **Cooldown / stabilization period**: wait after scaling to avoid **flapping** (scale up/down repeatedly).
- **Scale-out fast, scale-in slow**.
- **Warm-up time**: new instances take time to boot and warm caches. Use pre-baked images, containers, warm pools.
- **Graceful shutdown & connection draining** on scale-in.

**Good scaling metrics**: CPU, memory, requests/sec per instance, **queue depth** (for workers), latency p95/p99, custom business metrics. CPU alone can mislead (I/O-bound services).

**Where autoscaling applies**: VM groups (AWS ASG), containers (**Kubernetes HPA** for pods, **VPA** for pod size, **Cluster Autoscaler/Karpenter** for nodes), serverless (Lambda scales per request), databases (read replicas, serverless DB capacity), queue workers (KEDA).

**Cloud operational practices (what "good ops" looks like)**
1. **Infrastructure as Code (IaC)**: Terraform/CloudFormation, so environments are reproducible and version-controlled.
2. **CI/CD pipelines**: automated build, test, deploy.
3. **Deployment strategies**
   - **Rolling**: update a few instances at a time.
   - **Blue-Green**: two identical environments; switch traffic from old (blue) to new (green); instant rollback.
   - **Canary**: send 1–5% traffic to the new version, watch metrics, then expand.
   - **Feature flags**: deploy code dark, enable gradually, kill switch.
4. **Immutable infrastructure**: replace servers instead of patching them.
5. **Containers & orchestration**: Docker + Kubernetes for packaging, scheduling, self-healing.
6. **Multi-AZ** for HA; **multi-region** for disaster recovery.
7. **Observability** (logs/metrics/traces) and **alerting** with runbooks, on-call, post-mortems (blameless).
8. **Security**: least privilege IAM, secrets manager, encryption at rest/in transit, patching, VPC isolation.
9. **Cost management (FinOps)**: right-sizing, spot/preemptible instances for fault-tolerant jobs, reserved/savings plans for baseline, tagging, budgets.
10. **Backups & DR**: tested restores, RPO/RTO targets.
11. **Chaos engineering** and **game days**.
12. **Capacity & load testing** before big events.
13. **Quotas & limits** awareness (cloud account limits can block scaling!).

**Serverless**: you deploy functions; the provider scales automatically to zero and up. Pay per execution. Watch for cold starts, execution time limits, vendor lock-in, and downstream DB connection limits.

## 🎯 Analogy
Cinema counters opening and closing with queue length; cooldown = "don't fire staff after 2 minutes of calm"; canary = trying a new recipe on one table first.

## 🛠 Where it's used
E-commerce sales events, streaming spikes, batch processing, ride-hailing surge, SaaS with daily traffic patterns.

## ⭐ Important Points
- Autoscaling only works well with **stateless**, quickly-bootable services.
- It does **not** fix a slow database. Scaling app servers can *increase* DB pressure.
- Set sensible **max** to avoid runaway bills (and DDoS-driven cost explosions).
- Choose the metric that reflects **real saturation**.
- Slow scale-up vs sudden spikes → use scheduled/predictive scaling and over-provision buffer for known events.
- Always test rollback.

## ❓ Interview Q&A

**Q1. How does autoscaling work?**
A: A monitor collects metrics (CPU, RPS, queue depth); a policy defines thresholds or targets; when breached, the autoscaler launches/terminates instances within min/max limits, and the LB registers them after health checks pass.

**Q2. What's flapping and how do you avoid it?**
A: Rapid scale-out/in oscillation. Avoid with cooldowns, stabilization windows, hysteresis (different thresholds for in vs out), and slower scale-in.

**Q3. Reactive vs predictive vs scheduled scaling?**
A: Reactive responds to current metrics (lag); scheduled uses known patterns; predictive forecasts from historical data to scale before demand arrives.

**Q4. Blue-green vs canary deployments?**
A: Blue-green switches all traffic at once between two full environments (fast rollback, double cost during switch). Canary gradually shifts a small percentage to catch problems early with limited blast radius.

**Q5. What could go wrong when you autoscale the app tier?**
A: Database connection exhaustion, cache stampede from cold instances, downstream rate limits, slow warm-up, and cost spikes. Mitigate with connection pooling/proxies (e.g., RDS Proxy/PgBouncer), warm pools, limits, and backpressure.

**Q6. What is Infrastructure as Code and why use it?**
A: Defining infrastructure in version-controlled code. Gives reproducibility, review, automation, consistent environments, and easy disaster recovery.

**Q7. Which metric would you autoscale a queue worker fleet on?**
A: Queue depth (or age of oldest message) per worker, since it directly reflects backlog, rather than CPU.

**Q8. Serverless trade-offs?**
A: Pros: no server management, auto-scaling, pay-per-use. Cons: cold starts, time/memory limits, harder local debugging, lock-in, and possible high cost at constant heavy load.

---

# 14. Performance Engineering: Latency, Throughput & Monitoring

## 📖 Story
A courier company says, "Average delivery time: 2 days." Sounds good! But a customer's parcel took **12 days**. Averages hide pain. If 1 out of 100 parcels is badly late and you ship 10 million parcels, **100,000 people are angry**. Great performance engineers don't just look at averages; they look at the **slowest experiences** (p95, p99) and ask, "Where is the time going?"

## 🧠 Concepts

**Latency** = time to complete **one** request (ms).
**Throughput** = number of requests handled per unit time (RPS/QPS, MB/s).
**Bandwidth** = max data capacity of a link.
**Concurrency** = number of requests in flight.
**Little's Law**: `Concurrency = Throughput × Latency`. If you serve 1,000 RPS and each takes 200 ms (0.2 s), you have ~200 requests in flight.

> Latency and throughput are related but different: a highway with a high speed limit (low latency) can still jam (low throughput) with too few lanes.

**Percentiles (use these, not averages)**
- **p50** (median), **p90**, **p95**, **p99**, **p99.9**.
- **Tail latency**: the slowest requests; amplified in microservices (if a page calls 10 services, the chance that at least one is slow is much higher).

**Where latency comes from (a typical request)**
DNS → TCP/TLS handshake → network travel → LB/gateway → app code → cache/DB/other services → serialization → response travel → client rendering.

**Latency numbers every engineer should know (rough)**
| Operation | Approx. time |
|---|---|
| CPU cache / memory access | ~1–100 ns |
| SSD random read | ~100 µs |
| Same data-center round trip | ~0.5 ms |
| Read 1 MB from memory | ~0.25 ms |
| Read 1 MB from SSD | ~1 ms |
| Cross-continent round trip | 100–150 ms |
| Disk seek (HDD) | ~10 ms |
*Memory is fast, network is slow, disk is in between, and across oceans is slowest — so cache and place data near users.*

**Ways to reduce latency**
- Caching (browser, CDN, Redis), CDN, edge compute.
- DB indexing, query optimization, avoid N+1 queries, denormalize for reads.
- Connection pooling, keep-alive, HTTP/2/3.
- Compression (gzip/brotli), smaller payloads, pagination, field selection.
- Async/non-blocking I/O, offload slow work to queues.
- Parallelize independent calls; batch requests.
- Place services/data closer to users; avoid chatty cross-region calls.
- Precompute (materialized views), pre-warm caches.

**Ways to increase throughput**
- Horizontal scaling, load balancing, batching, efficient serialization (Protobuf), connection reuse, partitioning, backpressure to avoid collapse, removing locks/contention.

**Performance testing types**
- **Load test**: expected traffic. **Stress test**: beyond limits to find breaking point. **Spike test**: sudden surge. **Soak/endurance test**: long duration (finds memory leaks). **Capacity test**: how much can we handle. Tools: JMeter, k6, Gatling, Locust.

**Monitoring & observability**
- **Golden Signals (Google SRE)**: **Latency, Traffic, Errors, Saturation**.
- **RED method** (for services): Rate, Errors, Duration.
- **USE method** (for resources): Utilization, Saturation, Errors.
- **Three pillars**: **Metrics** (numbers over time: Prometheus/Grafana/CloudWatch), **Logs** (events: ELK/Loki/Splunk), **Traces** (request journey across services: Jaeger/OpenTelemetry/X-Ray).
- **Alerting**: alert on *symptoms users feel* (error rate, latency SLO burn), not just causes (CPU 80%). Reduce alert fatigue; attach runbooks.
- **SLI / SLO / SLA / Error budget**: SLI = measurement (e.g., % requests < 300 ms). SLO = internal target (99.9%). SLA = contract with penalties. Error budget = allowed failure (0.1%), which guides how fast you can ship.
- **APM & profiling**: find hot code paths (flame graphs), memory leaks, GC pauses.
- **Correlation IDs / trace IDs** across logs to follow one request everywhere.
- **Synthetic monitoring** (fake users probing) and **RUM** (real user monitoring).

**Backpressure & overload protection**: bounded queues, load shedding, timeouts, circuit breakers, graceful degradation. A system under overload should **fail fast and predictably**, not slowly and completely.

## 🎯 Analogy
Latency = how long one car takes to cross the bridge. Throughput = how many cars cross per minute. p99 = the unluckiest 1% of drivers. Distributed tracing = a GPS history of one car through every junction.

## 🛠 Where it's used
SLO-driven teams (Google, Netflix), e-commerce checkout optimization (every 100 ms costs conversions), trading, gaming, search.

## ⭐ Important Points
- **Averages lie; percentiles tell the truth.**
- Measure first, optimize the **biggest bottleneck** (don't guess).
- Tail latency grows in fan-out architectures; use timeouts, hedged requests, caching.
- Throughput and latency usually trade off: pushing utilization toward 100% makes latency explode (queueing). Keep headroom (~60–70%).
- Monitor the **four golden signals**; alert on user-impacting symptoms.

## ❓ Interview Q&A

**Q1. Latency vs throughput?**
A: Latency = time for one request. Throughput = how many requests per second. A system can have low latency but low throughput (single fast worker) or high throughput but high latency (batching).

**Q2. Why use p99 instead of average?**
A: Averages hide outliers. p99 shows what the slowest 1% experience. At scale, that 1% is thousands of users, and it often drives user-perceived quality.

**Q3. How would you debug a slow API?**
A: Check dashboards (latency percentiles, error rate, saturation), use distributed tracing to find the slow span (DB? downstream service? serialization?), examine slow query logs/`EXPLAIN`, check cache hit ratio, connection pool waits, GC pauses, recent deployments; then fix and verify with a load test.

**Q4. What are the golden signals?**
A: Latency, Traffic, Errors, Saturation.

**Q5. SLI, SLO, SLA?**
A: SLI = measured indicator, SLO = target on the SLI, SLA = contractual promise (with penalties) usually looser than the SLO.

**Q6. What is Little's Law?**
A: Concurrency = throughput × latency. Useful to size thread pools, connection pools, and capacity.

**Q7. How do you reduce tail latency?**
A: Timeouts, hedged/backup requests, caching, reducing fan-out, load shedding, avoiding GC/lock pauses, isolating noisy neighbors, and fixing slow dependencies.

**Q8. Metrics vs logs vs traces?**
A: Metrics = aggregated numeric trends (cheap, good for alerts). Logs = detailed discrete events (good for debugging). Traces = end-to-end journey of a request across services (find where time is spent).

**Q9. Why does latency spike as utilization nears 100%?**
A: Queueing theory: when arrival rate approaches service capacity, queues grow rapidly, so waiting time explodes. Keep spare capacity.

---

# 15. Putting It Together: Design a Product Catalog & Checkout

## 📖 Story
You're building **"ShopEasy"**, an online store. Millions of people browse products (lots of reading), but far fewer actually buy (fewer, very important writes). Browsing must be **fast**; checkout must be **correct**: no double charging, no selling the last item twice, no lost orders. Two different personalities in one system: **catalog = speed, checkout = correctness**.

## Step 1 — Requirements
**Functional**
- Browse/search/filter products, see details, images, price, stock.
- Add to cart, checkout, pay, get order confirmation, track order.
- Admin/seller: create/update products, inventory.

**Non-functional**
- Catalog reads: p99 < 200 ms, availability 99.99%.
- Checkout: strong correctness, no double charge, no oversell, 99.95% available.
- Scale: say 20M MAU, 2M DAU, peak 10× on sale days.
- Eventually consistent catalog is OK; **inventory/payment must be consistent**.

## Step 2 — Estimation (rough)
- 2M DAU × 30 product views/user = 60M views/day ≈ **~700 reads/sec avg**, peak ~7,000/s.
- Orders: 2% convert × 2M = 40K orders/day ≈ ~0.5/sec avg; sale-day peak maybe ~200/s.
- Read:Write ratio ≈ 100:1+ → **read-optimized catalog, careful write path.**
- Product data: 10M products × 5 KB = 50 GB (fits easily); images in object storage (TBs).

## Step 3 — High-Level Design
```
 Client (Web/Mobile)
    │  DNS (latency/geo routing)
    ▼
  CDN  ── static assets, images, cacheable catalog pages
    │
    ▼
  Load Balancer / API Gateway  (TLS, auth, rate limiting, routing)
    │
    ├── Catalog Service ──► Redis cache ──► Product DB (+ read replicas)
    │         └──► Search Service (Elasticsearch/OpenSearch)
    ├── Cart Service ──► Redis (fast, TTL) / DB
    ├── Inventory Service ──► Inventory DB (strong consistency)
    ├── Order Service ──► Orders DB (relational, ACID)
    ├── Payment Service ──► External PSP (Stripe/Razorpay) + Idempotency
    └── Notification Service ◄── Message Queue/Kafka ◄── events (OrderPlaced, PaymentDone…)
    
  Object Storage (S3) for images → served via CDN
  Observability: metrics, logs, tracing, alerts
```

## Step 4 — Catalog Design (optimize for reads)
**Data model (simplified)**
- `products(id, seller_id, name, description, category_id, price, currency, attributes JSON, status, updated_at)`
- `product_images(product_id, url, position)`
- `categories(id, parent_id, name)`
- Flexible attributes (color, size, RAM...) → JSON column or a document store (MongoDB/DynamoDB).

**API**
```
GET /v1/products?category=shoes&brand=nike&sort=-popularity&cursor=abc&limit=20
GET /v1/products/{id}
GET /v1/search?q=running+shoes
POST /v1/products            (seller/admin)
PATCH /v1/products/{id}
```
**Performance tactics**
1. **CDN** caches images and even product-detail JSON/pages with short TTL.
2. **Redis cache-aside** for product details (`product:{id}`); TTL + jitter; invalidate on update.
3. **Read replicas** for DB reads.
4. **Search** via Elasticsearch (inverted index, filters, facets, ranking), fed by **CDC/events** from the product DB (eventual consistency is fine).
5. **Cursor pagination**; return only needed fields.
6. **Denormalize** for read (store category name, primary image in the document).
7. **Popular products** can be pre-warmed; protect with stampede control.
8. **Price/stock display**: show cached "In stock / Low stock"; *authoritative check happens at checkout*.

## Step 5 — Cart
- Store in **Redis** (hash per user/session) with TTL (e.g., 30 days) for speed; persist to DB for logged-in users to survive cache loss.
- Merge guest cart into user cart on login.
- Cart does **not** reserve stock (usually); it just remembers intent.

## Step 6 — Checkout (optimize for correctness)
**Checkout flow**
1. **Validate cart**: re-fetch current prices/availability from authoritative sources (never trust client-sent prices!).
2. **Create Order** in status `PENDING` with a unique order ID + **idempotency key**.
3. **Reserve inventory** (temporary hold, e.g., 10-15 min with expiry).
4. **Payment**: call payment provider with idempotency key → get success/failure (often async via webhook).
5. On success: order `CONFIRMED`, inventory deducted permanently, publish `OrderConfirmed` event.
6. On failure/timeout: release inventory, order `FAILED/CANCELLED`.
7. Async consumers: send email/SMS, update analytics, trigger shipping, update search index.

**Preventing overselling (the last-item problem)**
- Option A: **Atomic conditional update** in the DB:
  `UPDATE inventory SET qty = qty - 1 WHERE sku = 'X' AND qty >= 1;` check rows-affected.
- Option B: **Optimistic locking** with a version column.
- Option C: **Pessimistic lock** (`SELECT ... FOR UPDATE`) — safer, lower concurrency.
- Option D: **Redis atomic counters** (DECR) for flash sales, reconciled with DB; or a queue that serializes purchases of hot items.
- Use **reservations with TTL** so abandoned carts don't block stock forever.

**Preventing double charge**
- Client generates an **Idempotency-Key** per checkout attempt; server stores key → result; repeated requests (double-click, retry) return the same result.
- Payment provider also supports idempotency keys.
- Make order creation **idempotent** (unique constraint on idempotency key).

**Distributed transaction across Order/Inventory/Payment → Saga pattern**
- Steps: ReserveInventory → ChargePayment → ConfirmOrder.
- If a step fails, run **compensating actions**: refund payment, release inventory, cancel order.
- Use **orchestration** (a coordinator service) or **choreography** (events).
- Use the **Outbox pattern**: write the order and the event to the same DB transaction; a relay publishes the event, so no "DB updated but event lost."

**Order state machine**: `PENDING → INVENTORY_RESERVED → PAID → CONFIRMED → SHIPPED → DELIVERED` (+ `CANCELLED/FAILED/REFUNDED`). Only valid transitions allowed.

**Databases choice**
- Catalog: document store or relational + cache + search.
- Orders/Inventory/Payments: **relational (ACID)**.
- Cart/session: Redis.
- Events: Kafka. Images: S3 + CDN. Search: Elasticsearch. Analytics: data warehouse.

## Step 7 — Scaling & Reliability
- Stateless services behind LB; autoscale on RPS/latency; separate scaling for catalog (read-heavy) vs checkout.
- **Sale-day spikes**: pre-scale, queue-based checkout ("virtual waiting room"), rate limit per user, cache aggressively, shed non-critical features.
- **DB**: primary + replicas, Multi-AZ failover; shard orders by `user_id` or `order_id` if needed.
- **Resilience**: timeouts, retries with backoff + jitter, circuit breakers around payment/inventory, bulkheads, fallback (e.g., show catalog without recommendations).
- **Multi-region** for reads (CDN + replicas); writes in a primary region or per-region ownership.
- **Security**: HTTPS, auth via JWT/OAuth, PCI-DSS (never store raw card data; use PSP tokens), input validation, CSRF/CORS, WAF, rate limiting on login/checkout, audit logs.
- **Observability**: dashboards for conversion funnel, checkout success rate, payment errors, inventory mismatches; alert on error budget burn; distributed tracing with order ID as correlation ID.
- **Data consistency/reconciliation**: nightly jobs to reconcile payments vs orders vs inventory.

## Trade-offs to mention
- Catalog: **eventual consistency** (cache, search lag) for speed.
- Checkout: **strong consistency** for money & stock.
- Reservation TTL: long = fewer failed payments but blocks stock; short = stock frees faster but users may lose their items.
- Sync vs async payment confirmation (webhooks + polling fallback).
- Monolith first vs microservices (team size, ops maturity).

## ❓ Interview Q&A

**Q1. How do you prevent two users from buying the last item?**
A: Make the stock decrement atomic: `UPDATE ... SET qty = qty-1 WHERE qty >= 1` (or optimistic locking). Reserve stock at checkout with an expiry, and confirm after payment. For flash sales, use a Redis atomic counter or queue to serialize hot-item purchases.

**Q2. How do you avoid double charging if the user clicks "Pay" twice or the network retries?**
A: Idempotency keys: the server stores the key and result; repeated calls return the stored result. Also unique constraints on order ID and the PSP's own idempotency support.

**Q3. Why separate catalog and checkout services?**
A: Their needs differ: catalog is read-heavy, cache-friendly, eventually consistent; checkout is write-sensitive, needs ACID and strict correctness. Separate scaling, storage, and failure isolation.

**Q4. How do you keep the search index in sync with the product DB?**
A: Publish change events (CDC/outbox → Kafka) and have an indexer consume them to update Elasticsearch. Accept slight lag; support periodic full reindex for repair.

**Q5. What happens if payment succeeds but the order service crashes?**
A: Use the Saga + outbox + idempotent consumers and reconciliation: the payment webhook is persisted and retried; on recovery the order is moved to CONFIRMED; a reconciliation job detects "paid but no order" and fixes/refunds.

**Q6. How do you handle a flash sale?**
A: Pre-scale, CDN + cache for the catalog, virtual waiting room/queue for checkout, per-user rate limits, atomic inventory counters, async order processing with clear status, and graceful degradation of non-critical features.

**Q7. Why not trust the price sent from the client?**
A: Clients can be tampered with. Prices, discounts, and stock must be recomputed server-side from authoritative data at checkout.

**Q8. Which database would you pick for each part?**
A: Products: document DB or relational + cache + search. Orders/payments/inventory: relational for ACID. Cart/session: Redis. Events: Kafka. Images: object storage + CDN.

**Q9. How do you show accurate stock on the product page?**
A: Show approximate stock from cache (e.g., "Only 3 left") for UX; the real check and reservation happens atomically at checkout. Tolerate slight staleness on the catalog.

---
# 🔶 Additional Topics (Missing From Your List But Essential)

---

# 16. Back-of-the-Envelope Estimation

## 📖 Story
Before building a bridge, an engineer estimates: "How many cars per hour? How heavy?" You don't need exact numbers, just the right **order of magnitude** to decide whether you need one lane or ten.

## 🧠 Concepts
**Useful numbers**
- 1 day = 86,400 s ≈ **~100K seconds** (use 10⁵ for quick math).
- 1M requests/day ≈ **~12 requests/sec**. 100M/day ≈ ~1,200/s.
- Powers: 2¹⁰ ≈ 1 thousand (KB), 2²⁰ ≈ 1 million (MB), 2³⁰ ≈ 1 billion (GB), 2⁴⁰ ≈ TB.
- 1 char ≈ 1 byte (ASCII). A tweet ≈ 300 bytes. A photo ≈ 200 KB–2 MB. A minute of HD video ≈ 50–100 MB.
- Peak traffic ≈ **2–10×** average.

**Formula toolkit**
- **QPS** = (DAU × actions per user per day) / 86,400.
- **Storage/day** = writes/day × size per record; **5-year storage** = ×365×5 (+ replication factor ×3).
- **Bandwidth** = QPS × average response size.
- **Servers needed** = peak QPS / QPS per server (add headroom 30–40%).
- **Cache size**: ~20% of daily hot data (80/20 rule).

**Worked example (URL shortener)**
- 100M new URLs/month → ~40 writes/s. Read:write = 100:1 → ~4,000 reads/s.
- Each record ~500 bytes → 100M × 500 B = 50 GB/month → ~3 TB over 5 years.
- Cache top 20% of daily reads: 4,000 × 86,400 × 0.2 × 500 B ≈ ~35 GB → fits in a Redis cluster.

## ⭐ Important Points
- State assumptions loudly; interviewers care about the **reasoning**, not exact numbers.
- Round aggressively; use powers of 10.
- Use estimates to *decide* (needs sharding? needs cache? one DB enough?).

## ❓ Interview Q&A
**Q1. Estimate QPS for 50M DAU, each making 10 requests/day.**
A: 500M requests/day ÷ 86,400 ≈ ~5,800 QPS average; peak ×3 ≈ ~17K QPS.

**Q2. Why do we estimate at all?**
A: To decide architecture: is a single DB enough? Do we need sharding, caching, CDN? How many servers and how much storage/bandwidth?

---

# 17. Databases: SQL vs NoSQL, Indexing

## 📖 Story
**SQL database** = a well-organized **Excel workbook** with strict columns, relations between sheets, and rules ("an order must belong to an existing customer"). **NoSQL** = a **box of flexible index cards or folders**: each card can have different fields, easy to spread across many boxes (servers).

**Index** = the index at the back of a book. Without it, you flip through every page (full table scan); with it, you jump to the page directly.

## 🧠 Concepts

**SQL / Relational (PostgreSQL, MySQL, Oracle)**
- Tables, rows, columns; **joins**; **ACID** transactions; strong schema.
- Great for structured data and relationships (orders, payments, inventory, banking).
- Scaling writes is harder (usually vertical + sharding by hand).

**ACID**
- **A**tomicity (all or nothing), **C**onsistency (valid state to valid state), **I**solation (concurrent transactions don't corrupt each other), **D**urability (committed = survives crash).

**Isolation levels** (weak → strong): Read Uncommitted → Read Committed → Repeatable Read → Serializable. Problems: dirty read, non-repeatable read, phantom read, write skew, lost update.

**NoSQL families**
| Type | Examples | Best for |
|---|---|---|
| Key-Value | Redis, DynamoDB, Riak | Sessions, caching, carts, simple lookups |
| Document | MongoDB, Couchbase | Catalogs, profiles, flexible JSON |
| Wide-column | Cassandra, HBase, ScyllaDB | Massive write throughput, time-series, IoT, messaging |
| Graph | Neo4j, Neptune | Relationships: social, fraud, recommendations |
| Search | Elasticsearch, OpenSearch | Full-text search, log analytics |
| Time-series | InfluxDB, TimescaleDB, Prometheus | Metrics |

**BASE** (NoSQL-ish): **B**asically **A**vailable, **S**oft state, **E**ventually consistent.

**How to choose**: data shape, query patterns, consistency needs, scale, team skill. *Start with a relational DB unless you have a clear reason not to.*

**Indexing**
- **B-Tree index** (default): good for equality & range queries (`WHERE age > 30`, `ORDER BY`). O(log n).
- **Hash index**: equality only.
- **Composite index**: `(country, city)` follows **leftmost prefix** rule.
- **Covering index**: index contains all needed columns, so no table lookup.
- **Clustered vs non-clustered**: clustered = data physically ordered by the key (one per table).
- **Inverted index**: word → list of documents (search engines).
- **LSM tree** (Cassandra, RocksDB): fast writes (append to memory + sorted files, compaction later). **B-tree**: faster reads, in-place updates.
- Trade-off: indexes speed **reads** but slow **writes** and use space. Don't index everything.
- Use `EXPLAIN` to see query plans; avoid functions on indexed columns in `WHERE`; fix **N+1** queries; avoid `SELECT *`.

**Normalization vs denormalization**
- Normalization: avoid duplication (clean, consistent writes, more joins).
- Denormalization: duplicate data for fast reads (fewer joins, harder updates). Common in NoSQL and read-heavy systems.

## ⭐ Important Points
- Money/inventory → relational ACID.
- Massive scale, simple access patterns → NoSQL.
- **Polyglot persistence**: use the right DB for each job.
- Design NoSQL schemas around **queries**, not around entities.

## ❓ Interview Q&A
**Q1. SQL vs NoSQL?**
A: SQL gives structure, joins, and ACID transactions; best for relational, consistency-critical data. NoSQL offers flexible schemas and easier horizontal scaling, typically trading joins/strong consistency for scale and speed. Choose by access pattern and consistency needs.

**Q2. How does an index make queries faster and what's the cost?**
A: It maintains a sorted structure (B-tree) so lookups are O(log n) instead of scanning all rows. Cost: extra storage and slower inserts/updates since indexes must be maintained.

**Q3. What is a composite index and the leftmost prefix rule?**
A: An index on multiple columns, e.g., `(country, city)`. It helps queries filtering by `country` or `country+city`, but not `city` alone.

**Q4. Explain ACID.**
A: Atomicity, Consistency, Isolation, Durability, guarantees for reliable transactions.

**Q5. When would you pick Cassandra?**
A: Huge write volume, time-series/event data, multi-region availability, known query patterns, and tolerance for eventual consistency.

**Q6. B-tree vs LSM-tree?**
A: B-tree: balanced reads, in-place updates. LSM: write-optimized (sequential writes, compaction), great for write-heavy loads but reads may check multiple files (mitigated by Bloom filters).

---

# 18. Replication, Sharding & Consistent Hashing

## 📖 Story
**Replication** = photocopies of the same notebook kept in several houses, so if one burns, you have others, and many friends can read at once.
**Sharding** = a huge encyclopedia split into volumes: A–C in volume 1, D–F in volume 2... Each shelf holds only part of the data, so no single shelf is overloaded.

## 🧠 Concepts

**Replication**
- **Leader–Follower (Primary–Replica)**: writes → leader; leader copies changes to followers; reads can go to followers.
  - **Synchronous**: leader waits for followers (safe, slower). **Asynchronous**: doesn't wait (fast, risk of data loss on failover, **replication lag**). **Semi-sync**: wait for at least one.
  - **Failover**: promote a follower when the leader dies (risk: split-brain, lost writes).
- **Multi-leader**: several writable nodes (multi-region writes); needs **conflict resolution** (last-write-wins, CRDTs, merge).
- **Leaderless (Dynamo-style)**: write to N nodes; use **quorum**: W + R > N ensures reads see the latest write (e.g., N=3, W=2, R=2).
- **Replication lag problems**: read-your-writes violation (you post, refresh, and don't see it). Fix: read from leader for recently written data, sticky reads, or session consistency.

**Sharding / Partitioning** (splitting data across nodes for write/storage scale)
- **Range-based**: by key ranges (A–M, N–Z). Good for range queries; risk of **hotspots**.
- **Hash-based**: `hash(key) % N`. Even distribution; poor range queries; adding nodes reshuffles almost everything.
- **Directory/lookup-based**: a map says where each key lives (flexible, extra hop).
- **Geo-sharding**: by region/tenant.
- **Choosing a shard key**: high cardinality, even distribution, matches common queries; avoid keys causing hotspots (e.g., timestamp, celebrity user ID).
- **Challenges**: cross-shard joins/transactions, **resharding**, hot shards ("celebrity problem"), operational complexity, global unique IDs.

**Consistent Hashing** ⭐
- Problem with `hash % N`: if N changes from 4 to 5, ~80% of keys move! (cache/DB meltdown).
- Solution: place nodes and keys on a **hash ring** (0 to 2³²-1). A key belongs to the **next node clockwise**.
- Adding/removing a node only moves keys from its neighbor (~1/N of data).
- **Virtual nodes (vnodes)**: each physical node appears at many ring positions → balanced load, smoother rebalancing, handles heterogeneous machines.
- Used by: DynamoDB, Cassandra, Memcached clients, CDNs, load balancers.

## 🎯 Analogy
Replication = photocopies; sharding = encyclopedia volumes; consistent hashing = a circular table where each person handles guests sitting up to the next person, so when someone leaves only their guests reshuffle.

## ⭐ Important Points
- Replication → **availability + read scale**. Sharding → **write + storage scale**. Often used together (each shard has replicas).
- Shard only when necessary; try indexes, caching, replicas, vertical scaling, and archiving first.
- Replication lag is a fact of life with async replication.
- Pick the shard key very carefully; it's hard to change later.

## ❓ Interview Q&A
**Q1. Replication vs sharding?**
A: Replication copies the same data to multiple nodes (HA, read scaling). Sharding splits different data across nodes (capacity, write scaling).

**Q2. Why consistent hashing?**
A: With simple modulo hashing, adding/removing nodes remaps most keys. Consistent hashing remaps only about 1/N keys, minimizing data movement and cache misses.

**Q3. What are virtual nodes?**
A: Multiple points on the ring per physical node to even out load and allow proportional capacity assignment.

**Q4. How do you handle a hot shard (celebrity user)?**
A: Split the hot key (key salting/sub-sharding), cache aggressively, replicate hot data, use dedicated shards, or separate handling for heavy entities.

**Q5. What does quorum mean (W + R > N)?**
A: Write to W nodes, read from R nodes among N replicas; if W+R>N the read set overlaps the latest write set, so reads see the latest committed write.

**Q6. Problems of async replication?**
A: Lag (stale reads) and potential data loss if the leader fails before replicating. Mitigate with semi-sync, read-from-leader for critical reads, monitoring lag.

---

# 19. CAP Theorem, Consistency Models & ACID/BASE

## 📖 Story
Two bank branches share one ledger by phone. Suddenly the phone line breaks (**network partition**). A customer asks branch A for the balance. Branch A can:
- **Refuse** to answer until the line is back (stays **consistent**, loses **availability**), or
- **Answer** with what it knows (stays **available**, may be **inconsistent**).
It cannot do both. That's CAP.

## 🧠 Concepts
**CAP Theorem**: In a distributed system, during a **network Partition (P)**, you must choose between **Consistency (C)** and **Availability (A)**.
- **CP**: consistent but may reject requests during partition (HBase, MongoDB (default), etcd, ZooKeeper, Spanner).
- **AP**: always responds, possibly stale data, reconciles later (Cassandra, DynamoDB, Couchbase).
- Partitions *will* happen, so real choice = C vs A **when partitioned**. When no partition, you trade **latency vs consistency** (**PACELC**).

**Consistency models (strong → weak)**
1. **Strong / Linearizable**: everyone sees the latest write immediately.
2. **Sequential**, **Causal** (cause precedes effect).
3. **Read-your-writes**, **Monotonic reads** (session guarantees).
4. **Eventual**: if no new writes, all replicas eventually converge.

**Conflict resolution**: last-write-wins (clocks!), vector clocks, CRDTs, app-level merge.

**Consensus**: Paxos, **Raft** (leader election + replicated log) allow nodes to agree despite failures; used in etcd, Consul, Kafka (KRaft). Needs a majority (quorum): 3 nodes tolerate 1 failure, 5 tolerate 2.

**Other**: **Two-Phase Commit (2PC)** for distributed transactions (blocking, coordinator failure issue); **Sagas** as an alternative.

## ⭐ Important Points
- CAP is about behavior **during partitions**, not a "pick any 2" menu.
- Different parts of one system can choose differently: payments = CP, product views/likes = AP.
- "Eventually consistent" doesn't mean "wrong," just "temporarily stale."

## ❓ Interview Q&A
**Q1. Explain CAP in simple words.**
A: When the network splits, a distributed system must either stay consistent (reject/wait) or stay available (respond with possibly stale data). It can't guarantee both.

**Q2. Is MySQL CP or AP?**
A: A single-node DB isn't really a distributed-system CAP case; with async replicas it behaves more AP for reads on replicas (stale) and CP-like on the primary. CAP labels are context-dependent.

**Q3. Give examples of where you'd choose AP vs CP.**
A: AP: social feed, likes, product catalog, shopping cart (Dynamo). CP: bank transfers, inventory counts, leader election, config stores.

**Q4. What is eventual consistency?**
A: Replicas may differ temporarily but converge when updates stop propagating, typically within ms–seconds.

**Q5. What is PACELC?**
A: If Partition: choose A or C; Else (normal operation): choose Latency or Consistency. It captures the everyday trade-off too.

---

# 20. Message Queues & Asynchronous Processing

## 📖 Story
At a busy restaurant, the waiter doesn't stand in the kitchen waiting for each dish. He drops the **order slip on a rail** (queue). Cooks pick slips at their own pace. If 50 orders arrive at once, they pile on the rail but aren't lost. Waiter and cooks are **decoupled**.

## 🧠 Concepts
**Message Queue / Broker**: stores messages from **producers** until **consumers** process them.
- **Queue (point-to-point)**: each message processed by one consumer (RabbitMQ, SQS). Work distribution.
- **Pub/Sub (topic)**: each message delivered to many subscribers (SNS, Kafka topics, Redis Pub/Sub).
- **Log-based streaming (Kafka)**: durable, ordered, partitioned log; consumers track offsets and can replay.

**Why use queues**: decoupling, **buffering spikes** (load leveling), async processing (emails, video encoding, reports), reliability via retries, scalability by adding consumers.

**Delivery guarantees**
- **At-most-once**: may lose messages.
- **At-least-once** ⭐ (most common): never lost, but may be duplicated → consumers must be **idempotent**.
- **Exactly-once**: very hard; usually "effectively once" = at-least-once + idempotent processing (Kafka transactions help in some cases).

**Key features**
- **Acknowledgements (ack)**: message removed only after consumer confirms processing; otherwise redelivered after visibility timeout.
- **Dead Letter Queue (DLQ)**: poison messages that keep failing are moved aside for inspection.
- **Retries with backoff**; **ordering** (guaranteed within a partition/key, not globally); **partitions** for parallelism; **consumer groups**; **backpressure**; **message TTL**; **priority queues**.
- **Kafka specifics**: topics → partitions → ordered logs; same key → same partition (ordering per key); retention by time/size; high throughput; replay.
- **Outbox pattern**: write business data + event in one DB transaction, then publish from outbox → avoids dual-write inconsistency.
- **Event-driven architecture**: services react to events (`OrderPlaced`). **Event sourcing**: store state as a sequence of events. **CQRS**: separate write model and read model.

**Kafka vs RabbitMQ/SQS (quick)**: Kafka = high-throughput event log, replay, streaming. RabbitMQ/SQS = classic task queues, flexible routing, simpler per-message semantics.

## 🛠 Where it's used
Email/SMS sending, order processing, video transcoding, notifications, analytics pipelines, log ingestion, CDC, fan-out feeds.

## ⭐ Important Points
- Async improves responsiveness: return `202 Accepted` quickly, finish work in background.
- **Idempotent consumers** are mandatory with at-least-once.
- Monitor **queue depth / consumer lag**.
- Queue growth = signal to scale consumers.
- Ordering only within a partition → choose the partition key wisely.

## ❓ Interview Q&A
**Q1. Why use a message queue?**
A: To decouple services, absorb traffic spikes, process work asynchronously, and add reliability through persistence and retries.

**Q2. At-least-once vs exactly-once?**
A: At-least-once guarantees delivery but may duplicate; exactly-once is hard in distributed systems, so we use at-least-once with idempotent consumers (e.g., dedupe by message ID).

**Q3. How to guarantee ordering?**
A: Put related messages (same user/order) on the same partition using a key; one consumer processes a partition sequentially. Global ordering limits scalability.

**Q4. What is a DLQ?**
A: A queue for messages that failed after N retries, so they don't block processing and can be investigated/reprocessed.

**Q5. Kafka vs RabbitMQ?**
A: Kafka is a distributed, durable log with replay and massive throughput, great for streaming/event pipelines. RabbitMQ is a flexible broker for task queues/complex routing with per-message acks.

**Q6. How do you handle a slow consumer?**
A: Scale out consumers (more partitions if needed), tune batch size, apply backpressure, alert on lag, and move failing messages to a DLQ.

---

# 21. Microservices vs Monolith & Resilience Patterns

## 📖 Story
A **monolith** is a **single giant shop** where billing, inventory, and delivery all work in one building. Simple at first; but one fire shuts everything, and all teams trip over each other. **Microservices** are **many small specialist shops** in a market, each with its own owner, so they can grow independently, but now you need roads, signboards, and rules for how they talk.

## 🧠 Concepts
| | Monolith | Microservices |
|---|---|---|
| Deploy | One unit | Independent |
| Scaling | Whole app | Per service |
| Complexity | In the code | In the network/ops |
| Data | One DB | DB per service |
| Team fit | Small teams | Many teams |
| Failure | Can take down all | Isolated (if designed well) |

> **Start with a well-structured (modular) monolith; split when pain justifies it.**

**Microservice essentials**: service discovery (Consul, K8s DNS), API gateway, config management, **database per service**, async events, **service mesh** (Istio/Linkerd: mTLS, retries, observability), centralized logging/tracing, CI/CD per service, contract testing, versioning.

**Resilience patterns**
- **Timeouts** (always!), **Retries with exponential backoff + jitter** (only for idempotent operations).
- **Circuit Breaker**: states **Closed** (normal) → **Open** (failing; calls blocked/fast-fail) → **Half-Open** (test a few calls). Prevents cascading failures.
- **Bulkhead**: isolate resource pools (thread pools/connection pools per dependency) like watertight ship compartments.
- **Fallback / graceful degradation**, **rate limiting**, **load shedding**, **backpressure**.
- **Health checks** (liveness vs readiness), **Sidecar** pattern, **Strangler Fig** pattern (migrate monolith piece by piece).
- **Saga** for cross-service transactions; **Outbox** for reliable events.
- **Distributed monolith** (anti-pattern): microservices that must be deployed together or share a DB.

## ⭐ Important Points
- Microservices trade code complexity for **operational/distributed complexity**.
- Each call over the network can fail or be slow, so design assuming that.
- Avoid chatty synchronous chains (A→B→C→D). Prefer async events where possible.
- Don't share databases across services.

## ❓ Interview Q&A
**Q1. When do you move from monolith to microservices?**
A: When team size, deployment contention, differing scaling needs, or domain boundaries make a single codebase a bottleneck, and when you have the ops maturity (CI/CD, observability, automation).

**Q2. Explain the circuit breaker pattern.**
A: It monitors calls to a dependency. After too many failures, it "opens" and fails fast (or uses a fallback) instead of waiting on timeouts; after a cool-off it goes "half-open" to test recovery, then closes if healthy.

**Q3. What's a bulkhead?**
A: Isolating resources per dependency so one slow dependency can't exhaust shared threads/connections and sink the whole service.

**Q4. Liveness vs readiness probes?**
A: Liveness: is the process alive (restart if not). Readiness: is it ready to take traffic (remove from LB if not).

**Q5. How do microservices communicate?**
A: Synchronously via REST/gRPC; asynchronously via events/queues. Prefer async for decoupling and resilience when immediate response isn't needed.

**Q6. What's the strangler fig pattern?**
A: Gradually replace parts of a monolith by routing specific functionality to new services behind a facade until the old system can be retired.

---

# 22. Idempotency & Distributed Transactions (Saga)

## 📖 Story
You press the lift button and nothing seems to happen, so you press it five times. A good lift doesn't come five times; it understands "same request." That's **idempotency**.
When booking a holiday, you need flight + hotel + cab. If the hotel fails after the flight is booked, you must **cancel the flight**. That's a **Saga**: a chain of steps, each with an "undo" step.

## 🧠 Concepts
**Idempotency**: performing an operation many times has the same effect as once.
- Natural: `PUT`, `DELETE`, "set status = PAID".
- Make non-idempotent ones safe using **idempotency keys**: client sends unique key; server stores `(key → response)`; duplicate key returns the stored response (use unique DB constraint or atomic `SET NX` in Redis with TTL).
- **Deduplication** in consumers: track processed message IDs.

**Distributed transactions**
- **2PC**: coordinator asks "prepare?" then "commit." Strong consistency but blocking, slow, and the coordinator is a SPOF. Rare in microservices.
- **Saga**: sequence of **local transactions** with **compensating transactions**.
  - **Choreography**: services react to each other's events (decentralized; harder to follow).
  - **Orchestration**: a central orchestrator tells each service what to do (clear flow; extra component).
  - Compensation example: `ChargePayment` ↔ `RefundPayment`; `ReserveStock` ↔ `ReleaseStock`.
  - Sagas are **eventually consistent**; design for intermediate states (`PENDING`).
- **Outbox pattern**, **Inbox pattern** (dedupe on consumer side), **Transactional messaging**.
- **TCC** (Try-Confirm-Cancel): reserve resources, then confirm or cancel.

## ⭐ Important Points
- Retries + at-least-once delivery = duplicates are *normal*. Idempotency is your safety net.
- Compensations must themselves be **idempotent and retryable**.
- Not all actions are reversible (email sent) → order steps so irreversible ones come last.

## ❓ Interview Q&A
**Q1. How would you implement an idempotent payment API?**
A: Require an `Idempotency-Key` header. On request, atomically insert the key with status `IN_PROGRESS` (unique constraint). If it exists, return the stored result (or 409 if still in progress). After completing, save the response against the key with a TTL.

**Q2. Saga vs 2PC?**
A: 2PC locks resources across services for atomic commit (strong consistency, poor availability/performance). Saga uses local transactions + compensations (eventual consistency, scalable, resilient).

**Q3. Orchestration vs choreography?**
A: Orchestration: central coordinator, easy to monitor, a single point of logic. Choreography: event-driven, decoupled, but harder to trace and can become tangled.

**Q4. What if a compensating action fails?**
A: Retry with backoff (idempotent), alert after limits, move to a manual-intervention queue, and reconcile periodically.

---

# 23. Object Storage, Search & Unique ID Generation

## 📖 Story
- **Object storage (S3)**: a **giant warehouse with labeled boxes**; you don't edit items in place, you drop/replace entire boxes, and the warehouse is nearly infinite and very durable.
- **Search (Elasticsearch)**: the **index at the back of a book** but for every word of every document.
- **Unique IDs**: every product, order, and tweet needs a **name tag that never repeats**, even when 100 machines are creating tags at once.

## 🧠 Concepts
**Object storage** (S3, GCS, Azure Blob): stores files (images, videos, backups, logs) as objects with a key + metadata. Cheap, durable (11 nines), scalable, HTTP API. Not for low-latency random updates or DB workloads.
- **Pre-signed URLs**: client uploads/downloads directly to S3 (don't pass big files through your servers).
- **Multipart upload** for large files; **lifecycle policies** (move to cold storage); versioning; serve via CDN.
- Block storage (disks for VMs/DBs), file storage (NFS), object storage (S3): know the differences.

**Search**
- **Inverted index**: `word → [doc ids]`. Supports full-text, ranking (TF-IDF/BM25), filters, facets, typo tolerance, autocomplete.
- Elasticsearch/OpenSearch/Solr. Feed it from the primary DB via CDC/events. It's a **secondary** store, not source of truth.
- **Autocomplete**: trie or prefix indexes, cached top suggestions.

**Unique ID generation in distributed systems**
| Approach | Pros | Cons |
|---|---|---|
| Auto-increment DB ID | Simple, ordered | Single point; hard to shard; guessable |
| UUID v4 | No coordination | 128-bit, random → poor index locality |
| UUID v7 / ULID | Time-ordered, unique | Larger than 64-bit |
| **Snowflake ID** | 64-bit, time-ordered, scalable | Needs machine ID & clock sync |
| Ticket server / range allocation | Simple, compact | Extra component |
Snowflake layout (Twitter): `1 bit sign | 41 bits timestamp | 10 bits machine ID | 12 bits sequence`.
**Base62 short codes** (for URL shorteners): encode the numeric ID in a–z, A–Z, 0–9.

**Other useful building blocks**: **Bloom filter** (probabilistic "definitely not present / maybe present", tiny memory), **HyperLogLog** (approximate unique counts), **Geohash/Quadtree** (location search), **Trie** (autocomplete), **Merkle tree** (data sync verification), **Heartbeat/gossip** (failure detection).

## ❓ Interview Q&A
**Q1. How do you let users upload a large video?**
A: Backend generates a pre-signed URL; client uploads directly to object storage with multipart upload; an event triggers async processing (transcoding) via a queue; results are stored and served via CDN.

**Q2. Why not use auto-increment IDs in a sharded system?**
A: Each shard would generate duplicates or need coordination; they're also guessable and a bottleneck. Use Snowflake/UUIDv7 or ID ranges.

**Q3. How does full-text search work?**
A: Text is tokenized and normalized into an inverted index mapping terms to documents; queries look up terms, intersect/rank results by relevance.

**Q4. What is a Bloom filter used for?**
A: Quickly checking whether an item *might* be in a set (no false negatives, some false positives) to avoid expensive lookups, e.g., cache penetration or DB lookups.

---

# 24. Security Essentials (HTTPS, OAuth, JWT, Common Attacks)

## 📖 Story
Think of your system as a bank: strong doors (network security), ID checks (auth), safes (encryption), CCTV (logging), and a rule that every employee only has keys for what they need (least privilege).

## 🧠 Concepts
- **Defense in depth**: multiple layers (WAF → gateway → service auth → DB permissions → encryption).
- **Encryption in transit** (TLS) and **at rest** (AES-256, KMS-managed keys). **Hashing passwords**: bcrypt/argon2/scrypt with salt, never plain or fast hashes.
- **Least privilege**, **zero trust** (never trust, always verify, even internal traffic).
- **Secrets management**: Vault/Secrets Manager; never in code/Git.
- **Common attacks & defenses**
  - **SQL injection** → parameterized queries/ORM.
  - **XSS** → output encoding, CSP.
  - **CSRF** → SameSite cookies, tokens.
  - **DDoS** → CDN/WAF, rate limiting, autoscaling, anycast.
  - **Brute force/credential stuffing** → rate limiting, MFA, lockouts, CAPTCHA.
  - **SSRF**, **IDOR** (accessing others' resources by changing an ID → always check ownership), **Man-in-the-middle** (TLS + HSTS), **Replay attacks** (nonce/timestamps), **Insecure deserialization**, **Dependency vulnerabilities** (scan, patch).
- **PII & compliance**: GDPR, PCI-DSS (never store raw card numbers; tokenize), data minimization, audit logs, data residency.
- **Input validation** everywhere; **API security**: authN/authZ per request, scopes, rate limits, schema validation.
- **RBAC vs ABAC**: role-based vs attribute-based access control.

## ❓ Interview Q&A
**Q1. How do you store passwords?**
A: Hash with a slow, salted algorithm like bcrypt/argon2. Never store plaintext or reversible encryption.

**Q2. How do you prevent IDOR?**
A: On every request verify the authenticated user is authorized for that specific resource, not just that they are logged in.

**Q3. Encryption at rest vs in transit?**
A: In transit protects data moving over networks (TLS); at rest protects stored data (disk/DB/object encryption with managed keys).

**Q4. How do you defend against DDoS?**
A: CDN/anycast absorption, WAF rules, rate limiting, autoscaling with budget caps, geo/IP blocking, and upstream scrubbing services.

---

# 25. Observability: Logs, Metrics, Traces, SLI/SLO/SLA

*(Core was covered in Topic 14; here are the extra operational pieces.)*

## 📖 Story
When a patient arrives in the ER, the doctor uses **vitals (metrics)**, **medical history notes (logs)**, and **scans following the problem through the body (traces)**. Without them, treatment is guesswork.

## 🧠 Concepts
- **Structured logging** (JSON with `trace_id`, `user_id`, `service`, `level`); central aggregation; sampling and retention policies; don't log secrets/PII.
- **Metrics types**: counters, gauges, histograms (for latency percentiles). Watch out for **high-cardinality labels** (cost explosion).
- **Distributed tracing**: context propagation via headers (`traceparent`/W3C Trace Context); **OpenTelemetry** is the standard.
- **Dashboards** for golden signals; **SLO-based alerting** (burn rate), **runbooks**, **on-call**, **incident management**, **blameless postmortems**.
- **Health endpoints**: `/health/live`, `/health/ready`.
- **Error budget policy**: if budget is exhausted → freeze features, focus on reliability.

## ❓ Interview Q&A
**Q1. How do you trace a request across 10 microservices?**
A: Generate a trace ID at the edge, propagate it via headers through every call and log line, and collect spans in a tracing backend (Jaeger/Tempo/X-Ray) to view the timeline.

**Q2. What's a good alert?**
A: Actionable, tied to user impact (error rate or latency SLO burn), with a runbook, minimal noise.

---

# 26. Disaster Recovery, Availability Math & Multi-Region

## 📖 Story
A museum keeps its masterpieces in the main hall, but also has **photographs in a vault in another city** and a **plan to reopen quickly** if the hall floods. How quickly you can reopen (**RTO**) and how recent the vault's copy is (**RPO**) decide how much it costs you.

## 🧠 Concepts
**Availability math**
- Series (all must work): `A = A1 × A2 × …` (two 99.9% parts in series → 99.8%).
- Parallel (redundant): `A = 1 − (1−A1)(1−A2)` (two 99% in parallel → 99.99%).
- Each extra nine costs much more. Dependencies lower your effective availability.

**RPO / RTO**
- **RPO (Recovery Point Objective)**: max acceptable **data loss** (time). "We can lose up to 5 minutes of data."
- **RTO (Recovery Time Objective)**: max acceptable **downtime**.

**DR strategies (cheap/slow → costly/fast)**
1. **Backup & Restore**: hours of RTO.
2. **Pilot Light**: minimal core always running in another region.
3. **Warm Standby**: scaled-down copy running.
4. **Multi-site Active-Active**: full capacity in multiple regions; near-zero RTO/RPO; most complex (data conflicts, cost).

**Multi-region concerns**: global traffic routing (DNS/anycast), data replication & conflict handling, latency, data residency laws, regional failover testing, cost.
**Practices**: automated backups with **tested restores**, cross-region replication, runbooks, game days, chaos testing, avoiding hidden single-region dependencies (auth, DNS, config).

## ❓ Interview Q&A
**Q1. RPO vs RTO?**
A: RPO = how much data you can afford to lose; RTO = how long you can be down.

**Q2. How would you design for region failure?**
A: Replicate data cross-region, keep infrastructure as code for quick rebuild or run warm/active-active, use health-checked DNS/global LB for failover, and rehearse regularly.

**Q3. Availability of two services in series, each 99.9%?**
A: ≈ 0.999 × 0.999 ≈ 99.8%. More dependencies in the critical path lower overall availability.

---

# 27. The Interview Framework & Final Cheat Sheet

## 📖 Story
A system design interview is like a **conversation with a senior architect on a whiteboard**. They're not checking if you memorized "Twitter's architecture"; they're checking **how you think**: do you ask questions, make reasonable assumptions, notice bottlenecks, and explain trade-offs?

## 🧭 The 7-step framework (≈ 45 minutes)
1. **Clarify requirements (5 min)**: features, users, scale, constraints, read/write ratio, consistency/latency/availability goals. Define what's *out of scope*.
2. **Estimate (3–5 min)**: DAU, QPS, storage, bandwidth.
3. **APIs (3–5 min)**: main endpoints, request/response.
4. **Data model (5 min)**: entities, relationships, DB choice, indexes, partition key.
5. **High-level design (10 min)**: boxes and arrows: client → CDN → LB/gateway → services → cache → DB → queue → workers.
6. **Deep dive (10–15 min)**: bottlenecks (hot keys, DB writes, fan-out), scaling, caching, consistency, failure handling.
7. **Wrap-up (3–5 min)**: trade-offs, monitoring, security, future improvements.

**Always say these out loud**: "The trade-off here is…", "If X fails, then…", "To scale this, I'd first measure…", "I'd start simple and evolve."

## 🧰 Quick decision cheat sheet
| Situation | Reach for |
|---|---|
| Slow reads | Index → cache → read replicas → CDN |
| Slow/spiky writes | Queue + async workers, batching, sharding |
| Global users | CDN, GeoDNS, multi-region reads |
| Need strict correctness | Relational DB, transactions, idempotency, locks |
| Huge scale, simple access | NoSQL, partitioning, eventual consistency |
| Long-running task | Queue + worker + status API (`202 Accepted`) |
| Real-time updates | WebSocket (2-way), SSE (server→client) |
| Cascading failures | Timeouts, retries+jitter, circuit breaker, bulkhead |
| Duplicate requests | Idempotency keys + dedupe |
| Hot key/hotspot | Local cache, key splitting, replication |
| Abuse/overload | Rate limiting, load shedding, WAF |
| Cross-service transaction | Saga + outbox |
| Search | Inverted index (Elasticsearch) fed by CDC |
| Big files | Object storage + pre-signed URL + CDN |
| Unique IDs at scale | Snowflake / UUIDv7 |

## 🔟 Common classic designs to practice
URL shortener • Rate limiter • Pastebin • Chat app (WhatsApp) • News feed/Twitter timeline • Notification system • YouTube/Netflix • Uber/ride matching • Search autocomplete • Web crawler • Distributed cache • Key-value store • Unique ID generator • Payment system • Ticket booking (BookMyShow) • Google Drive/Dropbox • Logging/metrics system • Hotel/flight booking • Food delivery (Zomato/Swiggy) • API gateway.

## ⚠️ Common mistakes to avoid
- Jumping into the design without clarifying requirements.
- Over-engineering (Kafka + microservices for 1,000 users).
- Ignoring failures, security, monitoring.
- Not stating trade-offs ("it depends" without saying *on what*).
- Using buzzwords without explaining why.
- Single points of failure left unaddressed.
- Forgetting data consistency in writes involving money/stock.
- Not quantifying: no numbers at all.

## 🎤 Top rapid-fire questions (one-line answers)
1. **Why stateless services?** Any instance can serve any request → easy scaling/failover.
2. **What reduces latency most cheaply?** Caching and moving data closer to users (CDN).
3. **How to scale DB reads?** Indexes, cache, read replicas.
4. **How to scale DB writes?** Sharding, queues, batching, write-optimized stores.
5. **Why consistent hashing?** Minimal key movement when nodes change.
6. **What is idempotency?** Same request repeated = same effect as once.
7. **CAP?** During partition choose consistency or availability.
8. **Why queues?** Decouple, absorb spikes, retry, async.
9. **L4 vs L7 LB?** Transport-level speed vs HTTP-aware routing.
10. **What is a circuit breaker?** Fail fast on a failing dependency to avoid cascading failure.
11. **p99 vs average?** p99 reveals the slow tail that average hides.
12. **Sticky sessions?** Pin user to a server; avoid, prefer shared session store.
13. **CORS?** Browser rule; server permits specific cross-origin reads.
14. **TCP vs UDP?** Reliable/ordered vs fast/best-effort.
15. **When NOT to use microservices?** Small team, unclear domain, low ops maturity.

---

## ✅ Final study plan (suggested)
- **Week 1**: Topics 1–6 (networking & infra) → draw the "request journey" from memory.
- **Week 2**: Topics 7–11 (APIs, security, comms, caching) → build a tiny REST API + Redis cache.
- **Week 3**: Topics 12–14, 16–19 (scaling, performance, databases, CAP) → practice estimations.
- **Week 4**: Topics 20–26 (queues, microservices, sagas, DR) → design 5 classic systems out loud.
- **Daily**: explain one concept to an imaginary friend using a story. If you can't explain it simply, you haven't mastered it yet.

> **Golden rule:** *Start simple. Make it work. Measure. Find the bottleneck. Scale that part. Explain every trade-off.* Good luck! 🚀
