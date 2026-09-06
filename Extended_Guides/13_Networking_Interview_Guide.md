# Networking Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Five-layer depth model: **One-liner · Analogy · Detailed Explanation · Scenario Context · Best Practices**

## How to Use This Guide
Q1–Q6 Junior; Q7–Q12 Mid-Level; Q13–Q16 Senior; Q17–Q20 production scenarios.

## SECTION A: JUNIOR LEVEL
### Q1. What is an IP address?
- **One-liner:** An IP identifies an interface on a network.
- **Analogy:** A postal address.
- **Detailed Explanation:** IPv4 uses 32 bits; IPv6 expands space; `ip addr` inspects interfaces.
- **Scenario Context:** A duplicate address causes intermittent reachability.
- **Best Practices:** Plan ranges and avoid public exposure.
### Q2. TCP versus UDP?
- **One-liner:** TCP provides reliable ordered connections; UDP is connectionless and lightweight.
- **Analogy:** Registered mail versus a live broadcast.
- **Detailed Explanation:** TCP handshake/retransmission; UDP suits DNS, streaming, and latency-sensitive traffic.
- **Scenario Context:** HTTP uses TCP/TLS; metrics may use UDP.
- **Best Practices:** Choose from correctness and latency requirements.
### Q3. What is DNS?
- **One-liner:** DNS maps names to records such as A, AAAA, CNAME, and SRV.
- **Analogy:** The internet phone book.
- **Detailed Explanation:** `dig api.example.com`; TTL and caching affect propagation.
- **Scenario Context:** Stale DNS sends clients to retired endpoints.
- **Best Practices:** Monitor expiry, health, and TTL.
### Q4. What is a subnet?
- **One-liner:** A subnet partitions an address range into a routing/security boundary.
- **Analogy:** A neighborhood within a city.
- **Detailed Explanation:** CIDR notation (`10.0.1.0/24`) defines network/host bits; route tables move traffic.
- **Scenario Context:** Private subnets lack direct Internet ingress.
- **Best Practices:** Reserve growth space.
### Q5. What is NAT?
- **One-liner:** NAT translates private addresses for selected connectivity.
- **Analogy:** A company receptionist using one public number.
- **Detailed Explanation:** SNAT enables egress; DNAT maps inbound traffic; NAT is not a security policy alone.
- **Scenario Context:** Private servers download patches through a NAT gateway.
- **Best Practices:** Restrict egress and monitor cost.
### Q6. What is TLS?
- **One-liner:** TLS authenticates endpoints and encrypts transport.
- **Analogy:** A sealed, signed conversation.
- **Detailed Explanation:** Certificates, trust chains, SNI, versions, and cipher suites matter; `openssl s_client` diagnoses handshakes.
- **Scenario Context:** Expired certificates cause client failures.
- **Best Practices:** Automate renewal and disable obsolete protocols.

## SECTION B: MID-LEVEL
### Q7. What is a load balancer?
- **One-liner:** It distributes traffic among healthy backends.
- **Analogy:** A dispatcher assigning customers to open counters.
- **Detailed Explanation:** L4 routes flows; L7 routes requests by host/path and can terminate TLS.
- **Scenario Context:** Health checks remove a failing instance.
- **Best Practices:** Probe meaningful readiness and preserve client IP deliberately.
### Q8. What is a reverse proxy?
- **One-liner:** It accepts client requests and forwards them to origin services.
- **Analogy:** A receptionist hiding the back office.
- **Detailed Explanation:** Nginx/Envoy can route, cache, terminate TLS, and enforce limits.
- **Scenario Context:** One public endpoint fronts several services.
- **Best Practices:** Set timeouts, headers, and access logs safely.
### Q9. What is BGP?
- **One-liner:** BGP exchanges reachability between autonomous systems.
- **Analogy:** Postal networks announcing delivery routes.
- **Detailed Explanation:** Prefixes, attributes, policy, and convergence affect path selection.
- **Scenario Context:** An incorrect advertisement blackholes traffic.
- **Best Practices:** Filter prefixes and monitor changes.
### Q10. What is a firewall?
- **One-liner:** A firewall permits or denies traffic using defined policy.
- **Analogy:** A security gate checking direction and badge.
- **Detailed Explanation:** Stateful/stateless rules, source/destination, ports, logging, and default deny matter.
- **Scenario Context:** Return traffic fails through a stateless ACL.
- **Best Practices:** Least privilege and reviewed rules.
### Q11. What is MTU?
- **One-liner:** MTU is the largest packet a link carries without fragmentation.
- **Analogy:** Maximum parcel size for a tunnel.
- **Detailed Explanation:** `ping -M do -s 1472 host` tests path MTU; overlays reduce effective size.
- **Scenario Context:** VPN connections hang on large responses.
- **Best Practices:** Document MTU and use MSS clamping where appropriate.
### Q12. How do you troubleshoot connectivity?
- **One-liner:** Walk layer by layer from DNS and route to policy, port, TLS, and application.
- **Analogy:** Check address, road, gate, door, and conversation.
- **Detailed Explanation:** Use `dig`, `ip route`, `traceroute`, `nc`, `curl -v`, flow logs, and packet capture.
- **Scenario Context:** A timeout differs from connection refusal.
- **Best Practices:** Capture source/destination/time and change one variable.

## SECTION C: SENIOR LEVEL
### Q13. Design a resilient network.
- **One-liner:** Use redundant zones/links, controlled routing, segmentation, and observable failover.
- **Analogy:** Multiple bridges with traffic controls.
- **Detailed Explanation:** Dual paths, health-based routing, private service access, firewalls, and tested DNS/route convergence compose resilience.
- **Scenario Context:** A link failure shifts traffic without exposing databases.
- **Best Practices:** Test real failure modes, not just ping.
### Q14. How do you segment services?
- **One-liner:** Separate trust zones and permit explicit flows between them.
- **Analogy:** Fire doors between building departments.
- **Detailed Explanation:** Subnets, security groups, network policy, identity-aware proxy, and egress controls reduce blast radius.
- **Scenario Context:** Web can reach API, not the database directly.
- **Best Practices:** Document flows and monitor denied traffic.
### Q15. Explain service discovery.
- **One-liner:** Discovery maps service identity to healthy changing endpoints.
- **Analogy:** A directory that updates when offices move.
- **Detailed Explanation:** DNS, registries, health checks, TTLs, and client retry behavior interact.
- **Scenario Context:** Stale endpoints cause retry storms.
- **Best Practices:** Set bounded retries and graceful degradation.
### Q16. How do you plan IP space?
- **One-liner:** Allocate non-overlapping hierarchical ranges for growth, connectivity, and policy.
- **Analogy:** Reserve city blocks before construction.
- **Detailed Explanation:** Account for clusters, VPN, peering, dual stack, and migrations; overlap complicates routing.
- **Scenario Context:** A merger fails because both networks use the same CIDR.
- **Best Practices:** Maintain an authoritative IPAM.

## SECTION D: PRODUCTION SCENARIOS
### Q17. An API times out after a network change.
- **One-liner:** Compare route, ACL, security policy, MTU, and backend health, then revert safely.
- **Analogy:** A road is open but a bridge or gate is not.
- **Detailed Explanation:** **What happened:** requests timeout after subnet route update. **Why:** return route was missing. **Fix:** 1) freeze change; 2) inspect routes/flow logs; 3) restore symmetric path; 4) test TCP/TLS; 5) validate; 6) add route test. **Prevention:** canary routing.
- **Scenario Context:** Timeouts commonly indicate dropped packets.
- **Best Practices:** Keep out-of-band access.
### Q18. DNS returns stale addresses.
- **One-liner:** Verify authoritative records, TTL/cache, resolver behavior, and endpoint health.
- **Analogy:** Old phone books still circulating.
- **Detailed Explanation:** **What happened:** clients hit retired nodes. **Why:** long TTL and incomplete record update. **Fix:** 1) `dig +trace`; 2) update authority; 3) flush only where controlled; 4) keep old endpoint during TTL; 5) verify cohorts; 6) shorten TTL. **Prevention:** automated DNS tests.
- **Scenario Context:** Do not assume local cache flush fixes public resolvers.
- **Best Practices:** Plan migrations around TTL.
### Q19. TLS certificates expire in production.
- **One-liner:** Renew, deploy the complete chain, verify all listeners, and automate expiry alerts.
- **Analogy:** Replace every access badge before its date.
- **Detailed Explanation:** **What happened:** clients reject expired certificate. **Why:** renewal job lacked permissions/alerting. **Fix:** 1) identify endpoints; 2) issue cert; 3) deploy chain; 4) test SNI/clients; 5) monitor; 6) review. **Prevention:** automated renewal and 30/7-day alerts.
- **Scenario Context:** A renewed leaf with missing intermediate still fails clients.
- **Best Practices:** Test from outside the network.
### Q20. A firewall change blocks a critical dependency.
- **One-liner:** Restore minimum required flow, identify rule cause, then codify and test policy.
- **Analogy:** A gatekeeper closed the only supply road.
- **Detailed Explanation:** **What happened:** database connections refused. **Why:** broad deny rollout. **Fix:** 1) stop propagation; 2) inspect logs; 3) restore narrow source/port rule; 4) verify app; 5) peer review; 6) staged rollout. **Prevention:** policy simulation and dependency inventory.
- **Scenario Context:** Confirm both directions for stateless controls.
- **Best Practices:** Default deny with tested exceptions.

## Coverage Summary
IP/CIDR, DNS, TCP/UDP, TLS, routing, NAT, load balancing, firewalls, MTU, segmentation, troubleshooting, and incidents.
