# Azure Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Five-layer depth model: **One-liner · Analogy · Detailed Explanation · Scenario Context · Best Practices**

## How to Use This Guide
Q1–Q6 Junior; Q7–Q12 Mid-Level; Q13–Q16 Senior; Q17–Q20 production scenarios.

## SECTION A: JUNIOR LEVEL
### Q1. What are Azure regions and availability zones?
- **One-liner:** Regions are geographic locations; zones are isolated datacenters within supported regions.
- **Analogy:** City and separated facilities.
- **Detailed Explanation:** Zone-redundant services and paired regions address different failures.
- **Scenario Context:** A zone failure should not remove all instances.
- **Best Practices:** Check service/region availability and sovereignty.
### Q2. What is a subscription and resource group?
- **One-liner:** A subscription scopes billing/limits; a resource group groups lifecycle-related resources.
- **Analogy:** Account ledger and project folder.
- **Detailed Explanation:** Azure Policy/RBAC can apply at management-group, subscription, RG, or resource scope.
- **Scenario Context:** Deleting a resource group can delete its contents.
- **Best Practices:** Separate production subscriptions and tag owners.
### Q3. Explain Azure RBAC.
- **One-liner:** RBAC assigns role permissions to identities at scopes.
- **Analogy:** A key issued for selected floors.
- **Detailed Explanation:** Entra ID users, groups, managed identities, built-in/custom roles, and deny assignments govern access.
- **Scenario Context:** A VM uses a managed identity to read Key Vault.
- **Best Practices:** Group-based least privilege and PIM.
### Q4. What is a virtual network?
- **One-liner:** VNet provides isolated IP space, subnets, routes, and security controls.
- **Analogy:** A campus with roads and guarded buildings.
- **Detailed Explanation:** NSGs filter traffic; private endpoints keep PaaS access private; VPN/ExpressRoute connect networks.
- **Scenario Context:** App subnet reaches database subnet without public exposure.
- **Best Practices:** Plan non-overlapping CIDRs and enable flow logs.
### Q5. What is Blob Storage?
- **One-liner:** Blob Storage stores scalable objects in storage accounts and containers.
- **Analogy:** A durable warehouse with labeled bins.
- **Detailed Explanation:** Access tiers, lifecycle, versioning, immutability, SAS, and encryption affect operation.
- **Scenario Context:** Logs move to cool/archive tiers after retention windows.
- **Best Practices:** Prefer managed identity over long-lived SAS.
### Q6. What is an Azure VM Scale Set?
- **One-liner:** VMSS manages a scalable set of similar VMs.
- **Analogy:** A standardized fleet with automatic hiring and replacement.
- **Detailed Explanation:** Images, instances, health probes, autoscale, zones, and rolling upgrades compose the fleet.
- **Scenario Context:** Failed instances are replaced automatically.
- **Best Practices:** Immutable images and multi-zone placement.

## SECTION B: MID-LEVEL
### Q7. Azure Load Balancer versus Application Gateway?
- **One-liner:** Load Balancer is Layer 4; Application Gateway adds Layer 7 routing/WAF.
- **Analogy:** A port dispatcher versus a receptionist reading requests.
- **Detailed Explanation:** Backend pools, probes, listeners, rules, TLS, and WAF policies determine behavior.
- **Scenario Context:** Path routing and WAF use Application Gateway.
- **Best Practices:** Probe real readiness and monitor backend health.
### Q8. What is AKS?
- **One-liner:** AKS is managed Kubernetes integrated with Azure identity/networking.
- **Analogy:** A managed control tower for container fleets.
- **Detailed Explanation:** Node pools, Entra integration, Azure CNI, managed identities, upgrades, and autoscaling matter.
- **Scenario Context:** Workload identity accesses Key Vault without secrets.
- **Best Practices:** Separate system/user pools and plan upgrades.
### Q9. What is Azure Functions?
- **One-liner:** Functions run event-driven code with managed scaling.
- **Analogy:** Call a specialist only when a task arrives.
- **Detailed Explanation:** Triggers/bindings, plans, cold starts, retries, and durable functions shape design.
- **Scenario Context:** Queue messages invoke idempotent workers.
- **Best Practices:** Set timeouts, DLQs, and concurrency carefully.
### Q10. What is Key Vault?
- **One-liner:** Key Vault stores secrets, keys, and certificates with controlled access.
- **Analogy:** A guarded vault issuing keys.
- **Detailed Explanation:** RBAC/access policies, managed identity, rotation, soft delete, purge protection, and diagnostics matter.
- **Scenario Context:** App retrieves a database secret at startup.
- **Best Practices:** Use managed identity and prevent accidental purge.
### Q11. How does Azure Monitor work?
- **One-liner:** Azure Monitor combines metrics, logs, alerts, and Application Insights.
- **Analogy:** Gauges, event archive, alarms, and application black box.
- **Detailed Explanation:** Log Analytics/KQL queries, diagnostic settings, workbooks, and action groups support operations.
- **Scenario Context:** KQL correlates failed requests with dependency calls.
- **Best Practices:** Centralize diagnostics and control retention.
### Q12. What is Azure Policy?
- **One-liner:** Policy evaluates and enforces organizational resource rules.
- **Analogy:** Building code checked continuously.
- **Detailed Explanation:** Definitions, initiatives, effects such as deny/audit/modify, and compliance assignments govern estates.
- **Scenario Context:** Policy denies public storage accounts.
- **Best Practices:** Start audit, remediate safely, then deny.

## SECTION C: SENIOR LEVEL
### Q13. Design an Azure landing zone.
- **One-liner:** Use management groups, subscriptions, identity, policy, hub-spoke networking, and central logging.
- **Analogy:** Corporate campus with guarded departments.
- **Detailed Explanation:** Entra, PIM, Azure Policy, Defender for Cloud, Firewall, private DNS, and Log Analytics establish foundations.
- **Scenario Context:** Workload subscriptions cannot disable central diagnostics.
- **Best Practices:** Automate subscription vending and exceptions.
### Q14. How do you choose AKS networking?
- **One-liner:** Choose kubenet or Azure CNI based on IP scale, routing, policy, and integration needs.
- **Analogy:** Decide whether vehicles use a private shuttle or campus roads directly.
- **Detailed Explanation:** Azure CNI consumes VNet IPs; kubenet conserves IPs but adds routing/NAT considerations.
- **Scenario Context:** Large clusters exhaust subnet IPs under CNI.
- **Best Practices:** Plan IP ranges and test upgrades.
### Q15. Explain Azure DR design.
- **One-liner:** Define RTO/RPO, replicate data, automate failover, and test paired/regional recovery.
- **Analogy:** A staffed duplicate facility with rehearsed transfer.
- **Detailed Explanation:** Site Recovery, geo-redundant storage, database replicas, Front Door/Traffic Manager, identities, and quotas must align.
- **Scenario Context:** DNS alone cannot recover a missing database.
- **Best Practices:** Run restore/failover exercises.
### Q16. How do you optimize Azure cost?
- **One-liner:** Attribute spend, right-size, schedule idle resources, and use commitments for baseline demand.
- **Analogy:** Fleet management by route and utilization.
- **Detailed Explanation:** Cost Management, Advisor, reservations/savings plans, spot VMs, storage tiers, and budgets help.
- **Scenario Context:** Orphaned disks remain after VM deletion.
- **Best Practices:** Cost per service/request and owner tags.

## SECTION D: PRODUCTION SCENARIOS
### Q17. AKS nodes are NotReady.
- **One-liner:** Protect workload capacity, inspect node conditions, and repair or replace the pool.
- **Analogy:** Remove a broken bus while preserving routes.
- **Detailed Explanation:** **What happened:** nodes lost heartbeat. **Why:** subnet/IP, kubelet, disk, or platform issue. **Fix:** 1) cordon; 2) inspect events/metrics; 3) validate networking/disk; 4) replace nodes; 5) check PDBs; 6) verify SLO. **Prevention:** zone pools and capacity alerts.
- **Scenario Context:** Do not drain all healthy capacity.
- **Best Practices:** Maintain tested node recovery.
### Q18. Key Vault access fails after deployment.
- **One-liner:** Verify identity, role scope, network path, secret version, and audit denial.
- **Analogy:** Badge, floor, doorway, and key must all work.
- **Detailed Explanation:** **What happened:** app receives 403. **Why:** managed identity lacked data-plane role/private DNS. **Fix:** 1) inspect Activity/diagnostic logs; 2) verify principal; 3) grant narrow role; 4) fix private endpoint DNS; 5) restart/read; 6) remove excess access. **Prevention:** IaC tests.
- **Scenario Context:** Control-plane permission is not data-plane permission.
- **Best Practices:** Avoid embedding secrets as fallback.
### Q19. A storage account was publicly reachable.
- **One-liner:** Disable public access, audit data, rotate credentials, and enforce policy.
- **Analogy:** Lock a warehouse and review who entered.
- **Detailed Explanation:** **What happened:** anonymous blob access enabled. **Why:** policy allowed legacy setting. **Fix:** 1) disable public access; 2) inspect logs; 3) revoke SAS; 4) notify; 5) enforce Policy; 6) validate clients. **Prevention:** deny public storage and scanner checks.
- **Scenario Context:** Check CDN/public website requirements before disabling.
- **Best Practices:** Separate public and private accounts.
### Q20. An Azure deployment caused an outage.
- **One-liner:** Stop rollout, restore last good version, diagnose dependency/config change, and add progressive gates.
- **Analogy:** Close the newly opened lane and reopen the safe lane.
- **Detailed Explanation:** **What happened:** App Service revision failed health checks. **Why:** incompatible setting/connection string. **Fix:** 1) swap back; 2) inspect Monitor/App Insights; 3) compare config; 4) fix in staging; 5) slot canary; 6) validate. **Prevention:** deployment slots and smoke tests.
- **Scenario Context:** Slots require schema-compatible changes.
- **Best Practices:** Keep rollback artifacts and logs.

## Coverage Summary
Azure hierarchy, IAM, networking, storage, compute, AKS, Functions, Key Vault, Monitor, Policy, DR, cost, and incidents.
