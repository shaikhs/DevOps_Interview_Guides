# Kubernetes Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Five-layer depth model: **One-liner · Analogy · Detailed Explanation · Scenario Context · Best Practices**

## How to Use This Guide
Q1–Q6 Junior; Q7–Q12 Mid-Level; Q13–Q16 Senior; Q17–Q20 production scenarios.

## SECTION A: JUNIOR LEVEL

### Q1. What is Kubernetes?
- **One-liner:** Kubernetes reconciles desired container workloads across a cluster.
- **Analogy:** An air-traffic controller schedules and replaces flights.
- **Detailed Explanation:** The API server stores objects; scheduler places Pods; controllers reconcile them; `kubectl get nodes` checks capacity.
- **Scenario Context:** A failed Pod is recreated automatically.
- **Best Practices:** Use declarative manifests and least privilege.
### Q2. What is a Pod?
- **One-liner:** A Pod is the smallest scheduled unit and shares network/storage among containers.
- **Analogy:** Containers are roommates sharing one address.
- **Detailed Explanation:** Pods are ephemeral; `kubectl describe pod X` shows events and status.
- **Scenario Context:** A sidecar shares localhost with its application.
- **Best Practices:** One main concern per Pod and explicit probes.
### Q3. Deployment versus ReplicaSet?
- **One-liner:** A Deployment manages rollout history; a ReplicaSet maintains Pod count.
- **Analogy:** Deployment is a release manager; ReplicaSet is the staffing roster.
- **Detailed Explanation:** `kubectl rollout status deploy/app`; Deployments create ReplicaSets for versions.
- **Scenario Context:** `rollout undo` returns to a prior ReplicaSet.
- **Best Practices:** Set revision history and rollout limits.
### Q4. What is a Service?
- **One-liner:** A Service gives stable virtual discovery and load balancing to changing Pods.
- **Analogy:** A switchboard forwards calls to available extensions.
- **Detailed Explanation:** ClusterIP is internal; NodePort/LoadBalancer expose externally; selectors choose endpoints.
- **Scenario Context:** Clients use `api.default.svc` rather than Pod IPs.
- **Best Practices:** Verify selectors and publish only required ports.
### Q5. ConfigMap and Secret?
- **One-liner:** ConfigMaps hold non-sensitive configuration; Secrets hold sensitive values with limited protection.
- **Analogy:** A noticeboard versus a locked envelope.
- **Detailed Explanation:** Mount or inject them; base64 is encoding, not encryption. `kubectl describe` can expose values indirectly.
- **Scenario Context:** A password should come from an external secret manager where possible.
- **Best Practices:** RBAC, encryption at rest, rotation, and no secrets in Git.
### Q6. What are requests and limits?
- **One-liner:** Requests guide scheduling; limits cap resource use.
- **Analogy:** Reserved seats versus a maximum luggage allowance.
- **Detailed Explanation:** CPU is compressible; memory over-limit can cause OOMKill. Inspect with `kubectl top pod`.
- **Scenario Context:** Missing requests cause noisy-neighbor and poor scheduling.
- **Best Practices:** Derive from measurements and use LimitRanges/quotas.

## SECTION B: MID-LEVEL
### Q7. Explain readiness, liveness, and startup probes.
- **One-liner:** Readiness controls traffic, liveness restarts unhealthy containers, startup protects slow boot.
- **Analogy:** Door-open status, vital signs, and time allowed to wake up.
- **Detailed Explanation:** Configure HTTP/TCP/exec probes with thresholds; bad liveness probes create restart loops.
- **Scenario Context:** Readiness fails during cache warmup without killing the process.
- **Best Practices:** Probe cheap, dependency-aware endpoints.
### Q8. How does scheduling work?
- **One-liner:** Scheduler filters feasible nodes then scores them using requests, affinity, taints, and policies.
- **Analogy:** Assigning cargo to vehicles meeting weight and route constraints.
- **Detailed Explanation:** `nodeSelector`, affinity, tolerations, topology spread, and priorities influence placement.
- **Scenario Context:** Databases spread across zones with anti-affinity.
- **Best Practices:** Avoid hard constraints that make recovery impossible.
### Q9. StatefulSet versus Deployment?
- **One-liner:** StatefulSet provides stable identity and ordered storage; Deployment suits interchangeable replicas.
- **Analogy:** Numbered apartments versus identical hotel rooms.
- **Detailed Explanation:** StatefulSet uses headless Services and PVC templates; termination/order have operational implications.
- **Scenario Context:** Kafka brokers need stable names and volumes.
- **Best Practices:** Test failover and backup application data.
### Q10. What is ingress?
- **One-liner:** Ingress expresses HTTP routing that an controller implements.
- **Analogy:** A receptionist routes visitors by host and path.
- **Detailed Explanation:** TLS, host/path rules, annotations, and controller behavior matter; Gateway API is newer and more expressive.
- **Scenario Context:** `/payments` routes to one Service and `/catalog` another.
- **Best Practices:** Standardize controller policy and certificate renewal.
### Q11. What is RBAC?
- **One-liner:** RBAC binds Roles or ClusterRoles to subjects for API authorization.
- **Analogy:** Keys granted to named job functions.
- **Detailed Explanation:** `kubectl auth can-i get pods --as user`; Role is namespace-scoped, ClusterRole can be cluster-wide.
- **Scenario Context:** CI can update Deployments only in one namespace.
- **Best Practices:** Avoid cluster-admin and audit authorization.
### Q12. How do rolling updates work?
- **One-liner:** A Deployment gradually replaces old Pods within availability and surge limits.
- **Analogy:** Replace buses one at a time while keeping service running.
- **Detailed Explanation:** `maxUnavailable`, `maxSurge`, readiness, and `kubectl rollout pause/resume` control rollout.
- **Scenario Context:** Failed readiness stalls rollout instead of routing bad Pods.
- **Best Practices:** Use canary/progressive delivery for high-risk changes.

## SECTION C: SENIOR LEVEL
### Q13. Design multi-tenant cluster isolation.
- **One-liner:** Combine namespaces, RBAC, quotas, network policies, admission, and node boundaries.
- **Analogy:** Offices share a building but have locks, budgets, and fire doors.
- **Detailed Explanation:** Namespaces alone are not isolation; enforce Pod Security, ResourceQuota, NetworkPolicy, and dedicated nodes for stronger boundaries.
- **Scenario Context:** A team cannot read another team’s Secrets or call its database.
- **Best Practices:** Use separate clusters/accounts for hostile or regulated tenants.
### Q14. Explain cluster upgrades.
- **One-liner:** Upgrade control plane, nodes, add-ons, and workloads with compatibility and rollback plans.
- **Analogy:** Replace a city’s traffic system section by section.
- **Detailed Explanation:** Read release notes, test APIs, drain nodes (`kubectl drain`), upgrade pools, validate metrics and workloads.
- **Scenario Context:** A removed API version breaks an old Helm chart.
- **Best Practices:** Back up etcd, rehearse, and maintain spare capacity.
### Q15. How do you operate Kubernetes at scale?
- **One-liner:** Standardize platform APIs, observability, policy, upgrades, and workload ownership.
- **Analogy:** A railway needs signals, timetables, maintenance, and station owners.
- **Detailed Explanation:** Use GitOps, admission policy, autoscaling, centralized logs/metrics/traces, and golden paths.
- **Scenario Context:** Teams self-serve namespaces without bypassing guardrails.
- **Best Practices:** Measure control-plane saturation and developer toil.
### Q16. Explain HPA, VPA, and Cluster Autoscaler.
- **One-liner:** HPA changes replicas, VPA recommends/changes requests, Cluster Autoscaler changes nodes.
- **Analogy:** Add workers, resize workers, or build another office.
- **Detailed Explanation:** HPA uses metrics; VPA restarts Pods for changes; node autoscaling depends on pending requests and cloud capacity.
- **Scenario Context:** HPA cannot help if requests are inflated or nodes have no capacity.
- **Best Practices:** Avoid competing controllers and test scale-down disruption.

## SECTION D: PRODUCTION SCENARIOS
### Q17. Pods are Pending after a release.
- **One-liner:** Read scheduler events, identify the constraint, then restore capacity or fix scheduling safely.
- **Analogy:** A truck cannot load because of weight, route, or no empty bay.
- **Detailed Explanation:** **What happened:** replicas remained Pending. **Why:** requests exceeded node capacity and topology rules. **Fix:** 1) `kubectl describe pod`; 2) inspect events/nodes; 3) correct requests or add nodes; 4) validate quotas/taints; 5) scale canary; 6) monitor. **Prevention:** capacity forecasts and admission validation.
- **Scenario Context:** Do not remove anti-affinity blindly.
- **Best Practices:** Alert on pending age.
### Q18. A rollout serves 5xx responses.
- **One-liner:** Stop rollout, route to healthy revision, diagnose readiness and app errors, then redeploy progressively.
- **Analogy:** Close a faulty lane and return traffic to the working lane.
- **Detailed Explanation:** **What happened:** new Pods became Ready but returned 500. **Why:** probe checked process, not dependency correctness. **Fix:** 1) pause/rollback; 2) inspect logs/events; 3) compare config; 4) improve readiness; 5) canary; 6) verify SLO. **Prevention:** synthetic readiness and contract tests.
- **Scenario Context:** Readiness must represent ability to serve.
- **Best Practices:** Keep rollback image cached/available.
### Q19. A Secret was exposed in a manifest.
- **One-liner:** Revoke and rotate first, then remove exposure and improve secret delivery.
- **Analogy:** Change a lock after publishing its key.
- **Detailed Explanation:** **What happened:** base64 Secret committed to Git. **Why:** encoding was mistaken for encryption. **Fix:** 1) rotate; 2) audit clones/logs; 3) remove history where policy requires; 4) use external Secrets/KMS; 5) restrict RBAC. **Prevention:** secret scanners and admission policy.
- **Scenario Context:** Assume the value is compromised even if the repository is private.
- **Best Practices:** Short-lived workload identity.
### Q20. A node failure causes data loss risk.
- **One-liner:** Preserve quorum, reschedule stateless work, restore state from replicated storage, and repair capacity.
- **Analogy:** Evacuate a damaged branch while protecting the ledger.
- **Detailed Explanation:** **What happened:** node died with a stateful Pod. **Why:** single replica/local storage. **Fix:** 1) cordon/drain healthy path; 2) inspect PVC and quorum; 3) restore/reattach replicated volume; 4) replace node; 5) verify consistency; 6) test restore. **Prevention:** replicas, topology spread, backups, and disruption budgets.
- **Scenario Context:** Never force-delete quorum members without understanding consensus.
- **Best Practices:** Run recovery drills.

## Coverage Summary
Kubernetes objects, networking, scheduling, security, scaling, upgrades, multi-tenancy, and incident response.
