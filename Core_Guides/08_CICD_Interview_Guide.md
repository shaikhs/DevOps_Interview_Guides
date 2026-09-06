# CI/CD Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Five-layer depth model: **One-liner · Analogy · Detailed Explanation · Scenario Context · Best Practices**

## How to Use This Guide
Q1–Q6 Junior; Q7–Q12 Mid-Level; Q13–Q16 Senior; Q17–Q20 production scenarios.

## SECTION A: JUNIOR LEVEL
### Q1. What is CI?
- **One-liner:** Continuous Integration frequently validates changes together.
- **Analogy:** Small puzzle pieces tested before the whole puzzle is assembled.
- **Detailed Explanation:** A pipeline checks out code, installs dependencies, tests, scans, and publishes evidence.
- **Scenario Context:** A broken commit fails before merge.
- **Best Practices:** Fast deterministic tests and protected branches.
### Q2. What is CD?
- **One-liner:** Delivery keeps software deployable; deployment automates release to environments.
- **Analogy:** A prepared meal versus serving it to customers automatically.
- **Detailed Explanation:** Promotion uses immutable artifacts, approvals, and rollback; delivery and deployment are not identical.
- **Scenario Context:** Main builds once and promotes the same image.
- **Best Practices:** Separate build from environment configuration.
### Q3. What is a pipeline stage?
- **One-liner:** A stage groups ordered jobs with inputs, outputs, and gates.
- **Analogy:** Airport security, boarding, and takeoff checkpoints.
- **Detailed Explanation:** Stages can run parallel or sequentially; artifacts and status flow between jobs.
- **Scenario Context:** Unit tests parallelize while deployment waits for all checks.
- **Best Practices:** Keep stages observable and restartable.
### Q4. Artifact versus source?
- **One-liner:** Source is input; an artifact is a versioned build output ready for deployment.
- **Analogy:** Raw ingredients versus sealed packaged goods.
- **Detailed Explanation:** Build once, checksum/sign, store in registry, promote by digest.
- **Scenario Context:** Production receives exactly what passed tests.
- **Best Practices:** Retain provenance and immutability.
### Q5. What is a runner/agent?
- **One-liner:** A runner executes pipeline jobs in a controlled environment.
- **Analogy:** A workshop assigned tasks from the production manager.
- **Detailed Explanation:** Hosted or self-hosted runners need tools, network, credentials, and cleanup.
- **Scenario Context:** A compromised job must not persist on the runner.
- **Best Practices:** Ephemeral least-privilege runners.
### Q6. Why cache dependencies?
- **One-liner:** Caching reduces build time but must not compromise correctness.
- **Analogy:** Keep commonly used tools nearby, but check versions.
- **Detailed Explanation:** Cache package directories keyed by lockfile; do not cache secrets or mutable outputs.
- **Scenario Context:** A lockfile hash invalidates stale dependencies.
- **Best Practices:** Measure hit rate and provide cache bypass.

## SECTION B: MID-LEVEL
### Q7. How do you promote artifacts?
- **One-liner:** Promote the same immutable artifact through environments, changing only controlled configuration.
- **Analogy:** One inspected product moves through warehouses.
- **Detailed Explanation:** Tag/digest, attest tests, environment approvals, and deploy without rebuilding.
- **Scenario Context:** Rebuild differences no longer explain prod-only bugs.
- **Best Practices:** Never use mutable `latest` as a release identity.
### Q8. What are deployment strategies?
- **One-liner:** Rolling, blue-green, canary, and feature flags control release exposure differently.
- **Analogy:** Replace a fleet, switch roads, test one vehicle, or hide a feature.
- **Detailed Explanation:** Select using rollback speed, capacity, state compatibility, and observability.
- **Scenario Context:** Canary catches elevated latency before all users see it.
- **Best Practices:** Define automated abort metrics.
### Q9. How do pipeline secrets work?
- **One-liner:** Inject short-lived scoped credentials at runtime and avoid logging them.
- **Analogy:** A temporary badge handed over at the worksite.
- **Detailed Explanation:** OIDC federation avoids stored cloud keys; masking is not perfect protection.
- **Scenario Context:** A job assumes a deployment role only for its account.
- **Best Practices:** Rotate and audit use.
### Q10. What is a quality gate?
- **One-liner:** A gate blocks promotion until required evidence meets policy.
- **Analogy:** A bridge inspection before opening traffic.
- **Detailed Explanation:** Gates may check tests, coverage, vulnerabilities, IaC policy, SLO health, or approval.
- **Scenario Context:** Critical CVEs stop an image from production.
- **Best Practices:** Make gates risk-based and actionable.
### Q11. How do you make pipelines reusable?
- **One-liner:** Templates/components standardize common jobs while exposing safe parameters.
- **Analogy:** A certified assembly line with configurable labels.
- **Detailed Explanation:** Shared workflows reduce duplication; version templates and prevent untrusted overrides.
- **Scenario Context:** Every service gets identical provenance and scanning.
- **Best Practices:** Treat pipeline libraries as production software.
### Q12. How do you observe CI/CD?
- **One-liner:** Track duration, queue time, failure causes, deployment frequency, and change failure rate.
- **Analogy:** Monitor both factory throughput and defect rate.
- **Detailed Explanation:** Emit job metrics, retain logs/artifacts, link commits to deployments and incidents.
- **Scenario Context:** Queue time reveals insufficient runners, not slow tests.
- **Best Practices:** Improve bottlenecks with data.

## SECTION C: SENIOR LEVEL
### Q13. Design a secure delivery platform.
- **One-liner:** Isolate runners, federate identity, sign artifacts, enforce policy, and maintain traceability.
- **Analogy:** A factory with controlled doors, inspection, and shipment manifests.
- **Detailed Explanation:** Use ephemeral runners, OIDC, SBOM/provenance, registry admission, protected environments, and audit logs.
- **Scenario Context:** Every production binary maps to source, builder, tests, and approver.
- **Best Practices:** Minimize trust between pipeline stages.
### Q14. How do you balance delivery speed and safety?
- **One-liner:** Automate low-risk checks and reserve humans for contextual/high-risk decisions.
- **Analogy:** Automatic brakes with a human driver for unusual roads.
- **Detailed Explanation:** Use progressive delivery, error budgets, policy tiers, and rollback automation.
- **Scenario Context:** A small internal change bypasses a manual gate; a database migration does not.
- **Best Practices:** Measure lead time alongside reliability.
### Q15. Explain database migrations in CD.
- **One-liner:** Make schema changes backward-compatible and deploy them in expand/migrate/contract phases.
- **Analogy:** Add a new road before removing the old road.
- **Detailed Explanation:** Expand columns/tables, deploy code supporting both, backfill safely, then contract after verification.
- **Scenario Context:** Rollback application code remains compatible with the expanded schema.
- **Best Practices:** Backups, locks/timeouts, and tested rollback.
### Q16. What is GitOps?
- **One-liner:** A declarative repository is the desired state, and an agent reconciles it.
- **Analogy:** A continuously checked blueprint rather than manual remote commands.
- **Detailed Explanation:** Pull-based agents apply reviewed commits and report drift; secrets and generated values need careful handling.
- **Scenario Context:** A manual cluster edit is reverted by reconciliation.
- **Best Practices:** Protect repos and define emergency break-glass procedure.

## SECTION D: PRODUCTION SCENARIOS
### Q17. A pipeline deployed the wrong commit.
- **One-liner:** Freeze promotion, identify artifact provenance, rollback by digest, and fix trigger ambiguity.
- **Analogy:** Stop shipping when the label does not match the box.
- **Detailed Explanation:** **What happened:** branch trigger used mutable tag. **Why:** checkout and build were not pinned to SHA. **Fix:** 1) halt; 2) identify deployed digest; 3) rollback; 4) pin SHA; 5) add provenance assertion; 6) retest. **Prevention:** immutable refs and deployment records.
- **Scenario Context:** Preserve logs before rerunning jobs.
- **Best Practices:** Display commit SHA prominently.
### Q18. CI credentials were leaked.
- **One-liner:** Revoke immediately, investigate access, and replace static credentials with federation.
- **Analogy:** Cancel a stolen badge before investigating doors opened.
- **Detailed Explanation:** **What happened:** secret appeared in logs. **Why:** debug output bypassed masking. **Fix:** 1) disable key; 2) inspect audit logs; 3) rotate downstream secrets; 4) purge artifacts; 5) patch job; 6) verify. **Prevention:** OIDC, secret scanning, and safe logging.
- **Scenario Context:** Treat logs as compromised copies.
- **Best Practices:** Least-privilege per job.
### Q19. Deployment is stuck halfway.
- **One-liner:** Stop further change, assess healthy capacity, and use a tested rollback or forward fix.
- **Analogy:** A bridge replacement paused with traffic on both sides.
- **Detailed Explanation:** **What happened:** half the fleet runs new code. **Why:** health gate and rollback were not wired. **Fix:** 1) pause; 2) inspect error budget; 3) drain/rollback unhealthy nodes; 4) repair; 5) canary; 6) complete. **Prevention:** atomic/progressive deployment and abort thresholds.
- **Scenario Context:** Ensure versions can coexist during partial rollout.
- **Best Practices:** Maintain last-known-good artifact.
### Q20. Tests are flaky and teams bypass CI.
- **One-liner:** Classify flake, reproduce, fix isolation, and track reliability as a pipeline SLO.
- **Analogy:** False fire alarms train people to ignore the siren.
- **Detailed Explanation:** **What happened:** intermittent failures led to retries/bypasses. **Why:** shared state and timing. **Fix:** 1) quarantine with owner; 2) capture seed/logs; 3) remove ordering/network dependence; 4) rerun under load; 5) restore gate; 6) monitor flake rate. **Prevention:** deterministic fixtures and budgets.
- **Scenario Context:** Retries should expose, not hide, instability.
- **Best Practices:** Never make bypass the normal path.

## Coverage Summary
CI/CD foundations, artifacts, runners, gates, release strategies, identity, GitOps, migrations, metrics, and incidents.
