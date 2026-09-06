# GCP Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Five-layer depth model: **One-liner · Analogy · Detailed Explanation · Scenario Context · Best Practices**

## How to Use This Guide
Q1–Q6 Junior; Q7–Q12 Mid-Level; Q13–Q16 Senior; Q17–Q20 production scenarios.

## SECTION A: JUNIOR LEVEL
### Q1. What are GCP regions and zones?
- **One-liner:** Regions contain zones; zones are independent failure domains.
- **Analogy:** City and separated buildings.
- **Detailed Explanation:** Deploy regional services across zones; `gcloud compute zones list` helps inventory.
- **Scenario Context:** A zonal failure should not remove all replicas.
- **Best Practices:** Match latency, sovereignty, and service availability.
### Q2. What is a project?
- **One-liner:** A project scopes resources, APIs, IAM, quotas, and billing.
- **Analogy:** A department ledger and workspace.
- **Detailed Explanation:** Enable APIs with `gcloud services enable`; use folders/org for hierarchy.
- **Scenario Context:** Production and development projects limit blast radius.
- **Best Practices:** Separate environments and budgets.
### Q3. Explain IAM roles.
- **One-liner:** IAM grants principals permissions through predefined or custom roles.
- **Analogy:** Job badges with allowed actions.
- **Detailed Explanation:** Bind roles to users, groups, or service accounts; deny broad inheritance.
- **Scenario Context:** Workload identity avoids service-account keys.
- **Best Practices:** Least privilege and audit logs.
### Q4. What is Compute Engine?
- **One-liner:** Compute Engine provides configurable virtual machines and disks.
- **Analogy:** Renting configurable servers.
- **Detailed Explanation:** Images, machine types, managed instance groups, and snapshots compose VM platforms.
- **Scenario Context:** MIG replaces unhealthy instances.
- **Best Practices:** Harden images and use regional redundancy.
### Q5. What is Cloud Storage?
- **One-liner:** Cloud Storage is durable object storage using buckets and objects.
- **Analogy:** A versioned warehouse.
- **Detailed Explanation:** `gcloud storage cp file gs://bucket`; lifecycle, retention, IAM, and encryption matter.
- **Scenario Context:** Build artifacts remain available after worker deletion.
- **Best Practices:** Uniform bucket-level access and public-access prevention.
### Q6. What is a VPC in GCP?
- **One-liner:** A VPC provides global virtual networking with regional subnets.
- **Analogy:** One campus network with buildings in different cities.
- **Detailed Explanation:** Routes, firewall rules, Cloud NAT, VPN/Interconnect, and private access connect workloads.
- **Scenario Context:** Private VMs reach updates through NAT without public IPs.
- **Best Practices:** Non-overlapping ranges and flow logs.

## SECTION B: MID-LEVEL
### Q7. What is a managed instance group?
- **One-liner:** MIGs maintain identical VM instances and support autoscaling/rolling updates.
- **Analogy:** A standardized fleet with automatic replacement.
- **Detailed Explanation:** Instance templates, health checks, autohealing, and regional distribution provide resilience.
- **Scenario Context:** A failed VM is recreated from a known image.
- **Best Practices:** Immutable templates and controlled rollouts.
### Q8. Cloud Load Balancing types?
- **One-liner:** GCP offers global/regional HTTP(S), TCP proxy, SSL proxy, and passthrough options.
- **Analogy:** Dispatchers choosing roads based on package type.
- **Detailed Explanation:** Backend services, health checks, URL maps, and NEGs determine routing.
- **Scenario Context:** Global HTTP load balancing sends users to healthy nearby backends.
- **Best Practices:** Test health paths and TLS policy.
### Q9. What is GKE?
- **One-liner:** GKE is managed Kubernetes with Google control-plane integration.
- **Analogy:** Kubernetes operations with a managed control tower.
- **Detailed Explanation:** Standard/autopilot modes, node pools, Workload Identity, and release channels define operation.
- **Scenario Context:** Autopilot handles nodes while teams manage Pods.
- **Best Practices:** Namespace/RBAC policy and upgrade planning.
### Q10. What is Cloud Run?
- **One-liner:** Cloud Run runs stateless containers with request-based autoscaling.
- **Analogy:** Open a service counter only when customers arrive.
- **Detailed Explanation:** Revisions, traffic splits, concurrency, min instances, and IAM control behavior.
- **Scenario Context:** 10% traffic tests a new revision.
- **Best Practices:** Set timeouts, concurrency, and authenticated ingress deliberately.
### Q11. Explain Pub/Sub.
- **One-liner:** Pub/Sub asynchronously delivers messages from publishers to subscriptions.
- **Analogy:** A broadcast station with subscriber mailboxes.
- **Detailed Explanation:** Ack deadlines, retries, ordering keys, retention, and dead-letter topics matter.
- **Scenario Context:** Consumers scale independently from publishers.
- **Best Practices:** Idempotence and backlog alerts.
### Q12. What is BigQuery?
- **One-liner:** BigQuery is managed analytical warehousing with columnar SQL execution.
- **Analogy:** A giant indexed archive optimized for questions.
- **Detailed Explanation:** Datasets, partitions, clustering, reservations, and IAM govern cost/performance.
- **Scenario Context:** Logs are queried without operating database servers.
- **Best Practices:** Partition filters and cost controls.

## SECTION C: SENIOR LEVEL
### Q13. Design a secure GCP landing zone.
- **One-liner:** Use organization/folders, separate projects, centralized logs, IAM, policies, and networking.
- **Analogy:** Corporate campus with guarded departments and central security.
- **Detailed Explanation:** Organization Policy, Cloud Identity, Shared VPC, SCC, Cloud Logging, and billing controls establish guardrails.
- **Scenario Context:** A workload project cannot disable central audit exports.
- **Best Practices:** Codify project vending and test policies.
### Q14. How do you choose GKE Autopilot versus Standard?
- **One-liner:** Autopilot reduces node operations; Standard provides deeper infrastructure control.
- **Analogy:** Managed taxi versus owning a fleet.
- **Detailed Explanation:** Compare privileged workloads, DaemonSets, networking, cost model, and operational skills.
- **Scenario Context:** Specialized GPU/node requirements favor Standard.
- **Best Practices:** Choose from workload constraints, not preference.
### Q15. Explain GCP disaster recovery.
- **One-liner:** Define RTO/RPO, replicate data, automate infrastructure, and test regional recovery.
- **Analogy:** Maintain a second staffed facility, not merely blueprints.
- **Detailed Explanation:** Use snapshots, dual-region storage, database replicas, DNS/traffic failover, and restore validation.
- **Scenario Context:** A region outage shifts stateless traffic while data recovery follows its runbook.
- **Best Practices:** Include IAM, quotas, secrets, and dependencies in drills.
### Q16. How do you control GCP cost?
- **One-liner:** Attribute, budget, optimize utilization, and govern expensive services.
- **Analogy:** Fleet fuel accounting by route.
- **Detailed Explanation:** Billing export, labels, Recommender, committed use, preemptible/spot VMs, BigQuery partitioning, and idle cleanup help.
- **Scenario Context:** Unpartitioned queries create unexpected spend.
- **Best Practices:** Cost per user/request and budget alerts.

## SECTION D: PRODUCTION SCENARIOS
### Q17. GKE Pods cannot schedule.
- **One-liner:** Inspect events, quotas, requests, taints, and node capacity, then restore safe capacity.
- **Analogy:** Cargo does not fit available trucks.
- **Detailed Explanation:** **What happened:** Pods Pending. **Why:** node pool limits and oversized requests. **Fix:** 1) describe Pod; 2) inspect nodes/quota; 3) correct requests or scale pool; 4) verify zones; 5) canary; 6) monitor. **Prevention:** capacity forecasts and quotas.
- **Scenario Context:** Do not remove safety constraints blindly.
- **Best Practices:** Alert on pending age.
### Q18. A Cloud Storage bucket became public.
- **One-liner:** Block access, audit reads, rotate exposed secrets, and add organization policy.
- **Analogy:** Close an unlocked warehouse and inspect inventory.
- **Detailed Explanation:** **What happened:** IAM allowed allUsers. **Why:** project policy lacked public-access prevention. **Fix:** 1) remove binding; 2) enable prevention; 3) inspect Data Access logs; 4) notify; 5) rotate; 6) validate. **Prevention:** SCC/policy-as-code.
- **Scenario Context:** Preserve object versions and logs.
- **Best Practices:** Separate public assets from sensitive buckets.
### Q19. A deployment revision has high latency.
- **One-liner:** Split traffic back, inspect Cloud Run metrics/traces, fix resource/concurrency, and retest.
- **Analogy:** Close a slow service lane.
- **Detailed Explanation:** **What happened:** new revision p99 rose. **Why:** concurrency exceeded dependency capacity. **Fix:** 1) reduce traffic; 2) inspect traces; 3) tune concurrency/min instances; 4) fix dependency; 5) canary; 6) verify. **Prevention:** load tests and automated rollback.
- **Scenario Context:** Cold starts and downstream limits differ.
- **Best Practices:** Monitor revision-level SLOs.
### Q20. Cross-project service account access fails.
- **One-liner:** Trace principal, IAM inheritance, service identity, and organization policy.
- **Analogy:** A badge works at headquarters but not the destination floor.
- **Detailed Explanation:** **What happened:** workload gets PermissionDenied. **Why:** missing target-project binding or Workload Identity mapping. **Fix:** 1) inspect audit event; 2) verify identity; 3) grant narrow role; 4) test read-only; 5) deploy; 6) audit. **Prevention:** Terraform IAM tests.
- **Scenario Context:** APIs may also need enabling and quota.
- **Best Practices:** Avoid service-account keys.

## Coverage Summary
GCP organization, IAM, networking, compute, storage, GKE, Cloud Run, Pub/Sub, BigQuery, DR, cost, and scenarios.
