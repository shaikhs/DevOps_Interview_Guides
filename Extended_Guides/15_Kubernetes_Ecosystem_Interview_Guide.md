# Kubernetes Ecosystem Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Five-layer depth model: **One-liner · Analogy · Detailed Explanation · Scenario Context · Best Practices**

## How to Use This Guide
Q1–Q6 Junior; Q7–Q12 Mid-Level; Q13–Q16 Senior; Q17–Q20 production scenarios.

## SECTION A: JUNIOR LEVEL
### Q1. What is Helm?
- **One-liner:** Helm packages Kubernetes manifests into versioned charts.
- **Analogy:** An application installation kit.
- **Detailed Explanation:** `helm template` renders; `helm upgrade --install` reconciles a release.
- **Scenario Context:** Values differ by environment.
- **Best Practices:** Pin charts and review rendered YAML.
### Q2. What is a CRD?
- **One-liner:** A CustomResourceDefinition extends the Kubernetes API with typed objects.
- **Analogy:** Add a new form type to city administration.
- **Detailed Explanation:** Controllers watch custom resources and reconcile them.
- **Scenario Context:** An operator manages databases through a CR.
- **Best Practices:** Version schemas and conversion carefully.
### Q3. What is an operator?
- **One-liner:** An operator encodes operational knowledge in a controller.
- **Analogy:** An automated specialist.
- **Detailed Explanation:** It watches desired state, acts, reports status, and handles upgrades/failures.
- **Scenario Context:** A database operator creates replicas and backups.
- **Best Practices:** Test idempotence and failure recovery.
### Q4. What is GitOps?
- **One-liner:** Git is the reviewed desired state reconciled by an agent.
- **Analogy:** A continuously checked blueprint.
- **Detailed Explanation:** Argo CD/Flux pull changes, compare drift, and report health.
- **Scenario Context:** A manual edit is reverted.
- **Best Practices:** Protect repos and define emergency access.
### Q5. What is a service mesh?
- **One-liner:** A mesh supplies traffic policy, identity, and telemetry between workloads.
- **Analogy:** Roads with signals and identity checkpoints.
- **Detailed Explanation:** Proxies provide mTLS, retries, routing, and metrics, at cost of complexity.
- **Scenario Context:** Canary by header.
- **Best Practices:** Prove the need before adoption.
### Q6. What is Kustomize?
- **One-liner:** Kustomize composes and patches plain manifests without templating.
- **Analogy:** Reusable stencils layered on a base drawing.
- **Detailed Explanation:** Bases, overlays, labels, and strategic patches generate environment output.
- **Scenario Context:** Production overlay changes replicas and domains.
- **Best Practices:** Validate rendered output.

## SECTION B: MID-LEVEL
### Q7. Helm release versus chart?
- **One-liner:** A chart is a package; a release is an installed chart revision.
- **Analogy:** Recipe versus a specific meal order.
- **Detailed Explanation:** History enables rollback; values and hooks can create risk.
- **Scenario Context:** `helm rollback` restores a prior release.
- **Best Practices:** Avoid destructive hooks and lock dependencies.
### Q8. How does Argo CD detect drift?
- **One-liner:** It compares Git desired state with live Kubernetes objects.
- **Analogy:** Blueprint inspector finding unauthorized remodeling.
- **Detailed Explanation:** Sync policies, health checks, waves, and diff customization control reconciliation.
- **Scenario Context:** An out-of-band replica change is corrected.
- **Best Practices:** Make exceptions explicit and auditable.
### Q9. What is Crossplane?
- **One-liner:** Crossplane exposes infrastructure resources through Kubernetes APIs.
- **Analogy:** One control desk for application and infrastructure requests.
- **Detailed Explanation:** Providers/controllers reconcile cloud resources and compositions create platform APIs.
- **Scenario Context:** Teams request a database claim without cloud credentials.
- **Best Practices:** Bound permissions and lifecycle ownership.
### Q10. What is a policy engine?
- **One-liner:** Admission policy validates or mutates objects before persistence.
- **Analogy:** Building inspector at the permit desk.
- **Detailed Explanation:** Kyverno/OPA can block privileged Pods, require labels, or verify signatures.
- **Scenario Context:** A deployment is rejected for missing resource limits.
- **Best Practices:** Audit first and provide remediation guidance.
### Q11. What is Prometheus Operator?
- **One-liner:** It manages monitoring resources such as ServiceMonitors and Prometheus servers.
- **Analogy:** A monitoring team automated through Kubernetes objects.
- **Detailed Explanation:** Labels select scrape targets; Alertmanager routes notifications.
- **Scenario Context:** A service adds metrics through a ServiceMonitor.
- **Best Practices:** Control cardinality and alert ownership.
### Q12. What is a CSI driver?
- **One-liner:** CSI connects Kubernetes storage APIs to external storage systems.
- **Analogy:** A universal adapter to different disk vendors.
- **Detailed Explanation:** StorageClasses provision PVCs; snapshots/clones and topology are driver-specific.
- **Scenario Context:** A zonal volume cannot attach in another zone.
- **Best Practices:** Test backup and restore with real drivers.

## SECTION C: SENIOR LEVEL
### Q13. Design a platform engineering stack.
- **One-liner:** Provide paved APIs, GitOps, policy, observability, upgrades, and ownership boundaries.
- **Analogy:** A reliable utility service for product teams.
- **Detailed Explanation:** Combine cluster lifecycle, Helm/Kustomize, operators, secret integration, policy, and developer portals.
- **Scenario Context:** Teams self-serve compliant services.
- **Best Practices:** Measure adoption, reliability, and toil.
### Q14. How do you govern operators?
- **One-liner:** Evaluate permissions, lifecycle, upgrade compatibility, support, and failure behavior.
- **Analogy:** Hire an automated contractor with a permanent key.
- **Detailed Explanation:** Review RBAC, CRD versions, finalizers, backups, webhooks, and resource consumption.
- **Scenario Context:** A broken webhook blocks all cluster writes.
- **Best Practices:** Isolate and canary operator upgrades.
### Q15. Explain progressive delivery.
- **One-liner:** Gradually expose versions using metrics-driven promotion and rollback.
- **Analogy:** Test one lane before opening a highway.
- **Detailed Explanation:** Argo Rollouts/Flagger use canary weights, analysis templates, and abort thresholds.
- **Scenario Context:** Error rate stops promotion at 10%.
- **Best Practices:** Define success metrics before rollout.
### Q16. How do you upgrade ecosystem components?
- **One-liner:** Map compatibility, stage changes, back up, and validate control-plane health.
- **Analogy:** Replace railway signals while trains still run.
- **Detailed Explanation:** Test CRDs, webhooks, CNI/CSI, ingress, metrics, and policy against target versions.
- **Scenario Context:** A CRD API removal breaks a controller.
- **Best Practices:** Maintain a compatibility matrix and rollback plan.

## SECTION D: PRODUCTION SCENARIOS
### Q17. GitOps repeatedly reverts an emergency fix.
- **One-liner:** Stabilize, record the fix in Git, and reconcile through the approved path.
- **Analogy:** The blueprint inspector keeps restoring the drawing.
- **Detailed Explanation:** **What happened:** live change was reverted. **Why:** Git is source of truth. **Fix:** 1) assess impact; 2) commit equivalent change; 3) review; 4) sync; 5) verify; 6) document break-glass. **Prevention:** emergency workflow.
- **Scenario Context:** Do not disable reconciliation casually.
- **Best Practices:** Time-bound pauses.
### Q18. An admission webhook blocks the cluster.
- **One-liner:** Isolate the webhook failure, restore safe availability, then repair policy.
- **Analogy:** A broken permit desk stops every building.
- **Detailed Explanation:** **What happened:** API requests timeout. **Why:** webhook service/cert/network failure. **Fix:** 1) inspect webhook and API events; 2) restore service/cert; 3) use scoped failure policy if safe; 4) validate; 5) repair; 6) test. **Prevention:** HA webhook and expiry alerts.
- **Scenario Context:** Never set fail-open for security-critical policy without approval.
- **Best Practices:** Keep a documented recovery path.
### Q19. Operator upgrade causes data risk.
- **One-liner:** Stop reconciliation, protect backups, restore compatible version, and test migration.
- **Analogy:** A new automated caretaker mishandles the archive.
- **Detailed Explanation:** **What happened:** CRD/controller changed ownership. **Why:** incompatible upgrade. **Fix:** 1) pause; 2) snapshot data; 3) inspect status/events; 4) rollback compatible image; 5) restore/test; 6) upgrade staged. **Prevention:** compatibility and restore tests.
- **Scenario Context:** Finalizers may block deletion; understand before removing.
- **Best Practices:** Follow vendor upgrade order.
### Q20. Mesh retry policy amplified an outage.
- **One-liner:** Stop retry amplification, restore dependency capacity, and tune bounded policies.
- **Analogy:** Every caller repeatedly redials a busy phone.
- **Detailed Explanation:** **What happened:** retries multiplied load. **Why:** unlimited retries/timeouts. **Fix:** 1) disable excess retries; 2) stabilize dependency; 3) set budgets/timeouts; 4) canary; 5) monitor; 6) document. **Prevention:** load tests and retry budgets.
- **Scenario Context:** Retries cannot create capacity.
- **Best Practices:** Prefer backoff and circuit breaking.

## Coverage Summary
Helm, operators, GitOps, Kustomize, policy, observability, storage, platform design, progressive delivery, upgrades, and incidents.
