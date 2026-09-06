# Container Ecosystem Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Every answer has **One-liner, Analogy, Detailed Explanation, Scenario Context, Best Practices**.

## How to Use This Guide
Q1–Q6 Junior; Q7–Q12 Mid-Level; Q13–Q16 Senior; Q17–Q20 production scenarios.

## SECTION A: JUNIOR LEVEL
### Q1. What is a container runtime?
- **One-liner:** Software that creates and runs containers.
- **Analogy:** An engine running standardized vehicles.
- **Detailed Explanation:** containerd/CRI-O use namespaces and cgroups; `ctr` or `crictl` inspects workloads.
- **Scenario Context:** Kubernetes delegates container execution to a runtime.
- **Best Practices:** Patch and harden the host/runtime.
### Q2. What is OCI?
- **One-liner:** OCI defines interoperable image and runtime specifications.
- **Analogy:** A common shipping-container standard.
- **Detailed Explanation:** Images and runtimes can move between compliant tools.
- **Scenario Context:** Build with BuildKit and run with containerd.
- **Best Practices:** Track formats and signatures.
### Q3. What is a registry?
- **One-liner:** A registry stores and distributes image manifests and layers.
- **Analogy:** A warehouse for versioned packages.
- **Detailed Explanation:** `docker pull registry/app@sha256:...`; auth, retention, and replication matter.
- **Scenario Context:** Clusters pull from a private registry.
- **Best Practices:** Immutable digests and access controls.
### Q4. What is BuildKit?
- **One-liner:** BuildKit is a modern efficient Docker build engine.
- **Analogy:** A factory that reuses work and parallelizes stations.
- **Detailed Explanation:** It supports cache mounts, secrets, multi-platform builds, and multi-stage output.
- **Scenario Context:** CI builds Linux/ARM images from one definition.
- **Best Practices:** Never bake secrets into layers.
### Q5. What are namespaces?
- **One-liner:** Kernel namespaces isolate process, network, mount, and identity views.
- **Analogy:** Separate windows into the same building.
- **Detailed Explanation:** They isolate visibility, not the shared kernel itself.
- **Scenario Context:** PID 1 inside a container differs from the host.
- **Best Practices:** Avoid privileged mode.
### Q6. What are cgroups?
- **One-liner:** Cgroups control and account for resource usage.
- **Analogy:** A utility meter and circuit breaker.
- **Detailed Explanation:** CPU, memory, I/O, and PID limits prevent noisy neighbors.
- **Scenario Context:** Memory pressure triggers an OOM kill.
- **Best Practices:** Set measured limits and alert.

## SECTION B: MID-LEVEL
### Q7. Docker Compose versus Podman Compose?
- **One-liner:** Both describe local multi-container apps; Podman emphasizes daemonless/rootless operation.
- **Analogy:** Similar recipes with different kitchen machinery.
- **Detailed Explanation:** Validate network, volume, and compatibility assumptions.
- **Scenario Context:** Developers run rootless integration stacks.
- **Best Practices:** Keep production orchestration separate.
### Q8. What is rootless containers?
- **One-liner:** Containers run without a privileged root daemon.
- **Analogy:** A tenant operating within their own lease.
- **Detailed Explanation:** User namespaces and subordinate IDs reduce host risk, with feature trade-offs.
- **Scenario Context:** A developer daemon escape has less impact.
- **Best Practices:** Test networking/storage compatibility.
### Q9. What is an image manifest list?
- **One-liner:** It maps one tag to platform-specific image manifests.
- **Analogy:** One product label with regional package versions.
- **Detailed Explanation:** `docker buildx build --platform linux/amd64,linux/arm64 --push` publishes multi-arch output.
- **Scenario Context:** ARM nodes pull the correct variant.
- **Best Practices:** Test every architecture.
### Q10. What are SBOMs and attestations?
- **One-liner:** They record contents and claims about how an image was built.
- **Analogy:** Parts list plus signed manufacturing certificate.
- **Detailed Explanation:** Generate CycloneDX/SPDX and provenance, then attach to digest.
- **Scenario Context:** Incident responders identify affected images quickly.
- **Best Practices:** Verify attestations at admission.
### Q11. What is a service mesh?
- **One-liner:** A mesh standardizes service-to-service traffic, identity, and telemetry.
- **Analogy:** A managed road system with tolls and traffic cameras.
- **Detailed Explanation:** Sidecars/ambient proxies provide mTLS, retries, routing, and metrics; they add complexity.
- **Scenario Context:** Canary traffic shifts by header.
- **Best Practices:** Adopt only for clear operational value.
### Q12. What is container storage?
- **One-liner:** Ephemeral layers suit stateless work; volumes provide persistence.
- **Analogy:** Disposable desk versus a permanent filing cabinet.
- **Detailed Explanation:** CSI/storage drivers expose block/file/object semantics; backup and restore are separate concerns.
- **Scenario Context:** Database data survives container replacement.
- **Best Practices:** Test performance and recovery.

## SECTION C: SENIOR LEVEL
### Q13. Design a container platform.
- **One-liner:** Standardize build, registry, runtime, policy, identity, observability, and support.
- **Analogy:** A safe industrial production line.
- **Detailed Explanation:** Golden images, signed artifacts, ephemeral workers, admission, quotas, and upgrade ownership define the platform.
- **Scenario Context:** Teams self-serve while platform controls risk.
- **Best Practices:** Publish SLOs and paved paths.
### Q14. How do you manage multi-architecture fleets?
- **One-liner:** Build/test manifests per architecture and schedule compatible workloads deliberately.
- **Analogy:** One recipe tested in different kitchens.
- **Detailed Explanation:** Native/cross builders, architecture labels, dependency compatibility, and performance testing matter.
- **Scenario Context:** An amd64-only binary fails on ARM.
- **Best Practices:** Declare platform constraints.
### Q15. Explain runtime hardening.
- **One-liner:** Reduce privileges, capabilities, syscalls, filesystem writes, and host exposure.
- **Analogy:** Give a worker a tool belt containing only required tools.
- **Detailed Explanation:** Non-root, seccomp, AppArmor/SELinux, read-only rootfs, dropped capabilities, and resource limits help.
- **Scenario Context:** A web process cannot modify host devices.
- **Best Practices:** Continuously scan and verify policy.
### Q16. How do you govern registries?
- **One-liner:** Govern identity, immutability, scanning, retention, replication, and promotion.
- **Analogy:** Warehouse receiving and shipment controls.
- **Detailed Explanation:** Separate write/read roles, quarantine, signatures, replication, and lifecycle rules.
- **Scenario Context:** Only approved digests reach production.
- **Best Practices:** Audit pushes and prune safely.

## SECTION D: PRODUCTION SCENARIOS
### Q17. Registry outage blocks deployments.
- **One-liner:** Use cached approved images, restore registry access, and improve availability.
- **Analogy:** Keep spare inventory when the warehouse closes.
- **Detailed Explanation:** **What happened:** pulls failed. **Why:** single registry endpoint. **Fix:** 1) pause rollout; 2) use cached/replica images; 3) restore auth/network; 4) verify digests; 5) resume canary; 6) add replication. **Prevention:** multi-region registry and cache.
- **Scenario Context:** Never substitute an unverified tag.
- **Best Practices:** Maintain last-known-good images.
### Q18. A privileged container is discovered.
- **One-liner:** Contain it, assess host exposure, replace with least privilege, and enforce admission.
- **Analogy:** A contractor received master keys.
- **Detailed Explanation:** **What happened:** `privileged: true` enabled host access. **Why:** workaround became permanent. **Fix:** 1) isolate; 2) audit host; 3) remove privilege; 4) test capabilities; 5) redeploy; 6) add policy. **Prevention:** Pod Security/admission checks.
- **Scenario Context:** Preserve forensic evidence.
- **Best Practices:** Time-bound exceptions.
### Q19. A multi-arch image fails on ARM.
- **One-liner:** Inspect manifest and binary architecture, rebuild/test the correct variant.
- **Analogy:** The label promised a vehicle that was not in the box.
- **Detailed Explanation:** **What happened:** exec format error. **Why:** amd64 binary published under a multi-arch tag. **Fix:** 1) inspect manifest; 2) build ARM; 3) test dependencies; 4) push digest; 5) canary; 6) verify nodes. **Prevention:** matrix CI.
- **Scenario Context:** Tags can hide per-platform differences.
- **Best Practices:** Deploy by verified digest.
### Q20. Container disk usage explodes.
- **One-liner:** Find logs/layers/volumes, reclaim safely, and set limits.
- **Analogy:** Warehouse overflow from uncollected packaging.
- **Detailed Explanation:** **What happened:** node filesystem filled. **Why:** unbounded logs and old layers. **Fix:** 1) inspect usage; 2) protect volumes; 3) rotate logs; 4) prune unreferenced layers; 5) restore headroom; 6) alert. **Prevention:** quotas and lifecycle.
- **Scenario Context:** Verify references before deleting.
- **Best Practices:** Externalize logs/state.

## Coverage Summary
OCI, runtimes, registries, builds, isolation, rootless security, multi-arch, meshes, storage, governance, and incidents.
