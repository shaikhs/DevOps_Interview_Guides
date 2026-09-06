# CI/CD Extended Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Five-layer depth model: **One-liner · Analogy · Detailed Explanation · Scenario Context · Best Practices**

## How to Use This Guide
Q1–Q6 Junior; Q7–Q12 Mid-Level; Q13–Q16 Senior; Q17–Q20 production scenarios.

## SECTION A: JUNIOR LEVEL
### Q1. What is pipeline-as-code?
- **One-liner:** Pipeline definitions live in version control.
- **Analogy:** The factory procedure is reviewed like product code.
- **Detailed Explanation:** YAML/workflows define triggers, jobs, artifacts, and gates.
- **Scenario Context:** A change is reviewed before execution.
- **Best Practices:** Pin actions and protect branches.
### Q2. What is a matrix build?
- **One-liner:** One job definition runs across versions/platforms.
- **Analogy:** Test a recipe in multiple kitchens.
- **Detailed Explanation:** OS/runtime matrices expose compatibility gaps.
- **Scenario Context:** Python versions are tested concurrently.
- **Best Practices:** Bound matrix size.
### Q3. What is an artifact cache?
- **One-liner:** Reusable dependency data shortens builds.
- **Analogy:** Keep tools near the workbench.
- **Detailed Explanation:** Key caches by lockfile and platform.
- **Scenario Context:** Dependency updates invalidate cache.
- **Best Practices:** Never cache secrets.
### Q4. What is a deployment environment?
- **One-liner:** A scoped target with configuration, approvals, and credentials.
- **Analogy:** A secured worksite.
- **Detailed Explanation:** Dev/stage/prod differ in access and gates, not artifact identity.
- **Scenario Context:** Production requires approval.
- **Best Practices:** Environment-specific OIDC roles.
### Q5. What is a webhook trigger?
- **One-liner:** An event starts automation.
- **Analogy:** Doorbell starts a workflow.
- **Detailed Explanation:** Validate signatures, event type, branch, and replay protection.
- **Scenario Context:** Only merged code deploys.
- **Best Practices:** Allowlist sources.
### Q6. What is rollback?
- **One-liner:** Return service to a known-good release.
- **Analogy:** Shift traffic back to the safe lane.
- **Detailed Explanation:** Roll back artifact/config while considering database compatibility.
- **Scenario Context:** Canary error rate aborts.
- **Best Practices:** Test rollback.

## SECTION B: MID-LEVEL
### Q7. What is blue-green deployment?
- **One-liner:** Two environments allow traffic switching between versions.
- **Analogy:** Open a second bridge before closing the first.
- **Detailed Explanation:** Validate green, shift traffic, retain blue for rollback.
- **Scenario Context:** DNS/load-balancer switch is fast.
- **Best Practices:** Budget duplicate capacity.
### Q8. What is canary deployment?
- **One-liner:** A small audience receives a release before wider promotion.
- **Analogy:** Taste one spoonful before serving.
- **Detailed Explanation:** Route by percentage/cohort and evaluate SLOs.
- **Scenario Context:** p99 rises at 5% and rollout stops.
- **Best Practices:** Define abort thresholds.
### Q9. What is feature flagging?
- **One-liner:** Runtime configuration controls feature exposure independently of deploy.
- **Analogy:** A light switch in a finished room.
- **Detailed Explanation:** Flags support cohorts, kill switches, and gradual release.
- **Scenario Context:** Disable a faulty path without rollback.
- **Best Practices:** Expire flags and audit access.
### Q10. What is a deployment freeze?
- **One-liner:** Temporarily restricts changes during high-risk periods.
- **Analogy:** Stop construction during a storm.
- **Detailed Explanation:** Emergency changes still need evidence and approval.
- **Scenario Context:** Holiday traffic freeze protects error budget.
- **Best Practices:** Define exit criteria.
### Q11. What is progressive delivery analysis?
- **One-liner:** Automated metrics decide promote, pause, or rollback.
- **Analogy:** A driving examiner controls the next route.
- **Detailed Explanation:** Compare baseline and candidate on errors, latency, saturation, and business outcomes.
- **Scenario Context:** Analysis fails after a dependency regression.
- **Best Practices:** Avoid noisy metrics.
### Q12. How do you manage migrations?
- **One-liner:** Make changes backward compatible and observable.
- **Analogy:** Replace plumbing while water still flows.
- **Detailed Explanation:** Expand/migrate/contract, backfill, throttle, and verify.
- **Scenario Context:** Old and new binaries coexist.
- **Best Practices:** Backup and rehearse.

## SECTION C: SENIOR LEVEL
### Q13. Design a global delivery platform.
- **One-liner:** Federate secure runners, artifact promotion, policy, rollout, and telemetry.
- **Analogy:** Global factories sharing quality standards.
- **Detailed Explanation:** Regional execution, provenance, OIDC, progressive delivery, and central evidence scale safely.
- **Scenario Context:** Region failure does not stop emergency fixes.
- **Best Practices:** Define ownership and SLOs.
### Q14. How do you handle pipeline governance?
- **One-liner:** Standardize controls while allowing bounded team autonomy.
- **Analogy:** Building codes with architectural freedom.
- **Detailed Explanation:** Reusable templates, policy checks, protected environments, and audit trails.
- **Scenario Context:** Teams cannot disable signing.
- **Best Practices:** Version templates and exceptions.
### Q15. What is continuous verification?
- **One-liner:** Production telemetry continuously validates release health.
- **Analogy:** Inspect the bridge while traffic crosses.
- **Detailed Explanation:** Compare release cohorts against SLO/business baselines and automate rollback.
- **Scenario Context:** Slow degradation triggers pause.
- **Best Practices:** Verify data quality.
### Q16. How do you recover CI/CD itself?
- **One-liner:** Keep pipeline definitions, runners, artifacts, and credentials recoverable.
- **Analogy:** A factory needs a backup control room.
- **Detailed Explanation:** Multi-region runners/registry, tested exports, break-glass, and immutable logs.
- **Scenario Context:** Primary CI outage still permits signed deployment.
- **Best Practices:** Exercise recovery.

## SECTION D: PRODUCTION SCENARIOS
### Q17. Canary metrics are inconclusive.
- **One-liner:** Pause, improve signal, and do not promote on missing evidence.
- **Analogy:** Do not open the bridge in fog.
- **Detailed Explanation:** **What happened:** telemetry gap. **Why:** sampling/label failure. **Fix:** 1) pause; 2) validate collectors; 3) compare logs/traces; 4) restore signal; 5) rerun canary; 6) verify. **Prevention:** analysis health gates.
- **Scenario Context:** No data is not success.
- **Best Practices:** Fail closed for promotion.
### Q18. Rollback fails because schema changed.
- **One-liner:** Stop traffic risk, use compatible forward fix or restore data safely.
- **Analogy:** Old key no longer fits changed lock.
- **Detailed Explanation:** **What happened:** old code cannot read new schema. **Why:** destructive migration. **Fix:** 1) assess; 2) restore compatible code; 3) add compatibility layer; 4) backup; 5) migrate; 6) verify. **Prevention:** expand/contract.
- **Scenario Context:** Application rollback and data rollback differ.
- **Best Practices:** Practice migration recovery.
### Q19. Runner compromise suspected.
- **One-liner:** Isolate runners, revoke credentials, investigate artifacts, and rebuild trusted infrastructure.
- **Analogy:** Close a contaminated factory.
- **Detailed Explanation:** **What happened:** unexpected outbound access. **Why:** persistent runner and broad token. **Fix:** 1) quarantine; 2) revoke; 3) preserve logs; 4) inspect outputs; 5) recreate ephemeral runners; 6) verify. **Prevention:** isolation and OIDC.
- **Scenario Context:** Treat artifacts as suspect.
- **Best Practices:** Minimal runner permissions.
### Q20. Release automation causes an outage.
- **One-liner:** Stop automation, restore known-good version, and add an approval/health gate.
- **Analogy:** A robot repeated a bad instruction.
- **Detailed Explanation:** **What happened:** bad config promoted globally. **Why:** no staged rollout. **Fix:** 1) halt; 2) rollback; 3) inspect blast radius; 4) repair config; 5) canary; 6) resume. **Prevention:** progressive delivery.
- **Scenario Context:** Preserve timeline and evidence.
- **Best Practices:** Rate-limit automation.

## Coverage Summary
Advanced pipeline design, promotion, canaries, flags, migrations, governance, verification, recovery, and delivery incidents.
