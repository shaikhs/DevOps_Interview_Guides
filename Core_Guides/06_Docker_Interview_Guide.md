# Docker Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Five-layer depth model: **One-liner · Analogy · Detailed Explanation · Scenario Context · Best Practices**

## How to Use This Guide
- Q1–Q6 Junior; Q7–Q12 Mid-Level; Q13–Q16 Senior; Q17–Q20 production scenarios.

## SECTION A: JUNIOR LEVEL

### Q1. What is an image versus a container?
- **One-liner:** An image is an immutable template; a container is a running instance with a writable layer.
- **Analogy:** A class recipe versus a meal cooked from it.
- **Detailed Explanation:** `docker build -t app:1 .` creates an image; `docker run --rm app:1` starts a container.
- **Scenario Context:** Multiple identical replicas come from one digest.
- **Best Practices:** Pin digests and keep containers disposable.
### Q2. What is a Dockerfile?
- **One-liner:** A Dockerfile declares reproducible image build steps.
- **Analogy:** A recipe with ordered ingredients and preparation.
- **Detailed Explanation:** `FROM`, `RUN`, `COPY`, `ENV`, `USER`, and `CMD` form layers and runtime defaults.
- **Scenario Context:** CI builds the same artifact across environments.
- **Best Practices:** Minimal base, non-root user, and no secrets in layers.
### Q3. Explain Docker layers and cache.
- **One-liner:** Each image instruction creates a content-addressed layer that can be reused.
- **Analogy:** Reusing unchanged pages in a photocopied manual.
- **Detailed Explanation:** Put stable dependency installs before changing source; inspect with `docker history`.
- **Scenario Context:** A poor `COPY .` early in the file invalidates all later cache.
- **Best Practices:** Use `.dockerignore` and deterministic builds.
### Q4. How do containers communicate?
- **One-liner:** Docker networks provide virtual connectivity and service discovery.
- **Analogy:** A private switch with named extensions.
- **Detailed Explanation:** `docker network create app`; containers on a user-defined network resolve names, while published ports expose host interfaces.
- **Scenario Context:** API connects to `db:5432` internally; only API publishes port 443.
- **Best Practices:** Avoid publishing databases and restrict network membership.
### Q5. What are volumes?
- **One-liner:** Volumes persist data outside a container’s writable layer.
- **Analogy:** A filing cabinet that remains when a temporary office is removed.
- **Detailed Explanation:** `docker volume create dbdata`; mount with `-v dbdata:/var/lib/postgresql/data`.
- **Scenario Context:** Replacing a database container does not erase its volume.
- **Best Practices:** Back up, encrypt, and verify restore.
### Q6. What do CMD and ENTRYPOINT do?
- **One-liner:** ENTRYPOINT defines the executable; CMD supplies default arguments or command.
- **Analogy:** A machine and its default job settings.
- **Detailed Explanation:** Exec form `ENTRYPOINT ["app"]` receives signals correctly; `docker run image --help` overrides CMD arguments.
- **Scenario Context:** Shell form may leave PID 1 unable to forward SIGTERM.
- **Best Practices:** Use exec form and graceful shutdown.

## SECTION B: MID-LEVEL
### Q7. How do you reduce image size?
- **One-liner:** Use multi-stage builds, slim bases, and remove build-only dependencies.
- **Analogy:** Ship the finished product, not the factory.
- **Detailed Explanation:** Build binaries in a builder stage, copy only artifacts into runtime; inspect with `docker image ls` and scanning.
- **Scenario Context:** A 1GB compiler image becomes a 40MB runtime image.
- **Best Practices:** Optimize only after measuring startup/security impact.
### Q8. What is a multi-stage build?
- **One-liner:** It separates compilation dependencies from the final runtime image.
- **Analogy:** A workshop stage followed by a clean storefront stage.
- **Detailed Explanation:** `FROM golang AS build`; then `FROM distroless`; `COPY --from=build /app/bin /app`.
- **Scenario Context:** Source and compilers never enter production.
- **Best Practices:** Reproducible pinned toolchains and non-root runtime.
### Q9. How should containers handle logs?
- **One-liner:** Write structured application logs to stdout/stderr and let the platform collect them.
- **Analogy:** Speak into the venue microphone rather than hiding notes backstage.
- **Detailed Explanation:** `docker logs` reads streams; configure drivers/agents and avoid unbounded files inside the container.
- **Scenario Context:** Central search correlates request IDs across replicas.
- **Best Practices:** JSON, redaction, retention, and rate limits.
### Q10. How do you secure Docker images?
- **One-liner:** Minimize attack surface, scan dependencies, sign artifacts, and run least privilege.
- **Analogy:** Ship a sealed package with a verified label and few tools inside.
- **Detailed Explanation:** Use Trivy/Grype, SBOMs, rootless where feasible, read-only filesystems, capabilities drops, and trusted registries.
- **Scenario Context:** CI blocks a critical OpenSSL vulnerability before promotion.
- **Best Practices:** Rebuild rather than patch live containers.
### Q11. What are resource limits?
- **One-liner:** CPU and memory limits prevent one container exhausting a host.
- **Analogy:** A utility meter prevents one tenant consuming the whole building.
- **Detailed Explanation:** `docker run --memory=512m --cpus=1 app`; observe OOM kills and throttling.
- **Scenario Context:** A memory leak is contained but causes restart churn.
- **Best Practices:** Set evidence-based requests/limits and alert on saturation.
### Q12. Docker Compose versus an orchestrator?
- **One-liner:** Compose coordinates local multi-container apps; orchestrators provide fleet scheduling and resilience.
- **Analogy:** A rehearsal stage versus a city-wide transport system.
- **Detailed Explanation:** Compose defines services, networks, and volumes; Kubernetes/Nomad add scheduling, service discovery, and reconciliation.
- **Scenario Context:** Compose runs integration tests; Kubernetes runs production replicas.
- **Best Practices:** Keep dev parity without pretending Compose is HA production.

## SECTION C: SENIOR LEVEL
### Q13. Design a secure image supply chain.
- **One-liner:** Build reproducibly, generate provenance/SBOM, scan, sign, and admit only trusted digests.
- **Analogy:** Factory quality control plus tamper-evident packaging.
- **Detailed Explanation:** Use isolated builders, pinned bases, attestations, cosign signatures, registry immutability, and admission policy.
- **Scenario Context:** Deployment verifies the digest and signature before scheduling.
- **Best Practices:** Patch base images continuously and track exceptions.
### Q14. Explain container isolation limits.
- **One-liner:** Containers share the host kernel, so they are process isolation, not virtual machines.
- **Analogy:** Separate apartments sharing a building’s foundation.
- **Detailed Explanation:** Namespaces isolate views; cgroups limit resources; capabilities/seccomp/SELinux reduce privilege. A kernel escape affects the host.
- **Scenario Context:** Untrusted workloads require stronger sandboxing or VMs.
- **Best Practices:** Harden hosts and avoid privileged containers.
### Q15. How do you choose a runtime strategy?
- **One-liner:** Match workload lifecycle, security, networking, storage, and operational maturity to the platform.
- **Analogy:** Choose a vehicle based on route, cargo, and driver skill.
- **Detailed Explanation:** Managed Kubernetes helps portability; serverless containers reduce ops; VMs may suit legacy/stateful workloads.
- **Scenario Context:** A batch worker does not need a service mesh.
- **Best Practices:** Optimize for reliability and operability, not trendiness.
### Q16. How do you debug a containerized service?
- **One-liner:** Check lifecycle, logs, config, network, resources, image, and dependencies in that order.
- **Analogy:** Diagnose a car from ignition to fuel to engine before replacing parts.
- **Detailed Explanation:** Use `docker inspect`, `logs`, `exec`, `stats`, `events`, health checks, and `docker network inspect`.
- **Scenario Context:** Exit code 137 indicates likely memory pressure, not necessarily an application bug.
- **Best Practices:** Capture immutable image digest and environment before changing anything.

## SECTION D: PRODUCTION SCENARIOS
### Q17. Containers repeatedly restart after deployment.
- **One-liner:** Identify exit reason and health behavior, then roll back or fix the image safely.
- **Analogy:** Determine whether a machine is crashing or being switched off by an alarm.
- **Detailed Explanation:** **What happened:** exit 1 followed rollout. **Why:** required environment variable was absent. **Fix:** 1) inspect `docker logs`/`inspect`; 2) compare config; 3) restore prior digest; 4) validate secret injection; 5) redeploy canary. **Prevention:** startup validation and config contract tests.
- **Scenario Context:** Distinguish crash loops from health-check restarts.
- **Best Practices:** Keep previous digests available.
### Q18. Host disk fills because of Docker.
- **One-liner:** Identify images, volumes, and JSON logs; reclaim only unreferenced data.
- **Analogy:** A warehouse filled with old boxes and receipts.
- **Detailed Explanation:** **What happened:** `/var/lib/docker` reached 100%. **Why:** unbounded logs and orphaned images. **Fix:** 1) `docker system df`; 2) cap/rotate logs; 3) remove unused objects after dependency check; 4) expand disk if needed; 5) verify service health. **Prevention:** retention and disk alerts.
- **Scenario Context:** Never prune volumes blindly; they may contain state.
- **Best Practices:** Externalize logs and schedule safe cleanup.
### Q19. A vulnerable base image is found in production.
- **One-liner:** Assess exploitability, rebuild from a patched base, scan, and roll out by digest.
- **Analogy:** Replace every batch from a recalled ingredient lot.
- **Detailed Explanation:** **What happened:** scanner found critical CVE. **Why:** stale pinned digest. **Fix:** 1) identify affected deployments; 2) assess exposure; 3) rebuild/test; 4) sign and promote; 5) canary/replace; 6) revoke old artifact. **Prevention:** rebuild automation and SLA.
- **Scenario Context:** Prioritize internet-facing and exploitable paths.
- **Best Practices:** Record accepted-risk expiry, not permanent exceptions.
### Q20. A container cannot reach its database.
- **One-liner:** Validate DNS, network membership, port policy, credentials, and database health.
- **Analogy:** Check address, road, gate, key, and destination in order.
- **Detailed Explanation:** **What happened:** API times out to `db`. **Why:** containers were on separate networks. **Fix:** 1) inspect networks; 2) test DNS/port; 3) attach correct network; 4) verify SG/firewall and credentials; 5) add health/readiness dependency. **Prevention:** integration tests and explicit network definitions.
- **Scenario Context:** A DNS failure differs from a refused connection.
- **Best Practices:** Use service names, not ephemeral container IPs.

## Coverage Summary
- Images, builds, runtime, networking, storage, security, supply chain, debugging, and production recovery.
