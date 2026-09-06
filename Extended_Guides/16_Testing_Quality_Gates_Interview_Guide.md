# Testing & Quality Gates Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Five-layer depth model: **One-liner · Analogy · Detailed Explanation · Scenario Context · Best Practices**

## How to Use This Guide
Q1–Q6 Junior; Q7–Q12 Mid-Level; Q13–Q16 Senior; Q17–Q20 production scenarios.

## SECTION A: JUNIOR LEVEL
### Q1. Unit test?
- **One-liner:** Tests one small unit in isolation.
- **Analogy:** Check one gear before the machine.
- **Detailed Explanation:** Mock boundaries; run quickly in CI.
- **Scenario Context:** A function regression fails pre-merge.
- **Best Practices:** Deterministic focused tests.
### Q2. Integration test?
- **One-liner:** Tests collaboration with a real dependency or subsystem.
- **Analogy:** Test connected gears.
- **Detailed Explanation:** Use disposable databases/containers and migrations.
- **Scenario Context:** SQL mapping passes unit tests but fails integration.
- **Best Practices:** Isolate data and clean up.
### Q3. End-to-end test?
- **One-liner:** Validates a user journey across the deployed system.
- **Analogy:** Test the whole factory order.
- **Detailed Explanation:** Browser/API flow covers routing, auth, services, and persistence.
- **Scenario Context:** Checkout smoke catches broken wiring.
- **Best Practices:** Keep few high-value E2E tests.
### Q4. What is test coverage?
- **One-liner:** Coverage measures executed code/branches, not correctness.
- **Analogy:** Count rooms visited, not whether inspected well.
- **Detailed Explanation:** Line/branch coverage informs gaps; mutation testing tests assertion strength.
- **Scenario Context:** 95% coverage still misses timeout behavior.
- **Best Practices:** Never optimize the number alone.
### Q5. What is a quality gate?
- **One-liner:** A rule that blocks unsafe promotion.
- **Analogy:** Bridge inspection before opening.
- **Detailed Explanation:** Tests, security, lint, policy, and performance evidence can gate.
- **Scenario Context:** Critical vulnerability blocks release.
- **Best Practices:** Make failure actionable.
### Q6. What is a flaky test?
- **One-liner:** A test with inconsistent results for the same code.
- **Analogy:** A smoke alarm that rings randomly.
- **Detailed Explanation:** Timing, shared state, order, network, and randomness are common causes.
- **Scenario Context:** Retries hide a race.
- **Best Practices:** Track, quarantine with owner, then fix.

## SECTION B: MID-LEVEL
### Q7. Test pyramid?
- **One-liner:** Many fast unit tests, fewer integration, few E2E tests.
- **Analogy:** Wide foundation, narrow roof.
- **Detailed Explanation:** Optimize feedback and confidence by test cost/risk.
- **Scenario Context:** A pipeline taking hours needs lower-level coverage.
- **Best Practices:** Choose tests by failure mode.
### Q8. Contract testing?
- **One-liner:** Verifies provider/consumer interface compatibility independently.
- **Analogy:** Both sides agree on a shipping label.
- **Detailed Explanation:** Consumer contracts validate fields/statuses; provider verifies them.
- **Scenario Context:** API removes a field used by a client.
- **Best Practices:** Version contracts and test negative cases.
### Q9. Mutation testing?
- **One-liner:** Deliberately changes code to see whether tests detect it.
- **Analogy:** Introduce defects to test the inspector.
- **Detailed Explanation:** Surviving mutants reveal weak assertions; use selectively.
- **Scenario Context:** A test passes despite removed authorization.
- **Best Practices:** Run on critical modules, not every commit.
### Q10. Performance testing?
- **One-liner:** Measures behavior under load, stress, spike, and soak conditions.
- **Analogy:** Load-test a bridge before rush hour.
- **Detailed Explanation:** Define workload, latency percentiles, throughput, saturation, and acceptance thresholds.
- **Scenario Context:** p99 rises at expected traffic.
- **Best Practices:** Production-like data and safe environments.
### Q11. Security testing?
- **One-liner:** Finds weaknesses through static, dependency, dynamic, and abuse testing.
- **Analogy:** Inspect blueprint, parts, and finished building.
- **Detailed Explanation:** SAST/SCA/DAST/fuzzing complement threat modeling.
- **Scenario Context:** DAST finds an auth bypass missed by SAST.
- **Best Practices:** Prioritize exploitable findings.
### Q12. Test data management?
- **One-liner:** Provide reliable, safe, representative data for tests.
- **Analogy:** Practice patients with realistic but anonymous records.
- **Detailed Explanation:** Fixtures, factories, masking, synthetic data, and cleanup avoid leakage/order dependence.
- **Scenario Context:** Production PII must not enter CI.
- **Best Practices:** Reset state between tests.

## SECTION C: SENIOR LEVEL
### Q13. Design a testing strategy.
- **One-liner:** Map risks to layered tests, environments, gates, and observability.
- **Analogy:** Security checks at every stage of a journey.
- **Detailed Explanation:** Balance speed, confidence, cost, contract compatibility, performance, and recovery tests.
- **Scenario Context:** A payment service needs stronger controls than a static page.
- **Best Practices:** Use risk-based evidence.
### Q14. What makes a gate trustworthy?
- **One-liner:** It is deterministic, relevant, fast enough, owned, and hard to bypass.
- **Analogy:** A reliable airport checkpoint.
- **Detailed Explanation:** Pin dependencies, isolate tests, version policy, expose evidence, and audit overrides.
- **Scenario Context:** Teams trust a gate that explains failure.
- **Best Practices:** Measure false positives and escape rate.
### Q15. Explain release verification.
- **One-liner:** Confirm deployment health through smoke, metrics, logs, traces, and business checks.
- **Analogy:** Inspect the vehicle after leaving the factory.
- **Detailed Explanation:** Automated checks compare baseline/error budget and trigger rollback.
- **Scenario Context:** Deployment is technically successful but checkout fails.
- **Best Practices:** Verify user outcomes, not process exit code.
### Q16. How do you test resilience?
- **One-liner:** Inject controlled failures and verify recovery against RTO/RPO/SLO.
- **Analogy:** Fire drills with measured evacuation.
- **Detailed Explanation:** Test dependency loss, node/zone failure, latency, restore, and capacity exhaustion.
- **Scenario Context:** Circuit breaking prevents cascading failure.
- **Best Practices:** Start small, schedule, and stop safely.

## SECTION D: PRODUCTION SCENARIOS
### Q17. Flaky tests block releases.
- **One-liner:** Classify and fix flake; do not normalize retries.
- **Analogy:** Repair the unreliable alarm.
- **Detailed Explanation:** **What happened:** intermittent CI failures. **Why:** shared state/race. **Fix:** 1) capture seed/logs; 2) reproduce; 3) isolate fixtures; 4) fix synchronization; 5) remove quarantine; 6) monitor. **Prevention:** flake budget.
- **Scenario Context:** Retain evidence from failed runs.
- **Best Practices:** Owner and deadline for quarantine.
### Q18. Quality gate passed but production failed.
- **One-liner:** Find coverage gap, restore service, and add a representative test.
- **Analogy:** Inspector checked the wrong bridge.
- **Detailed Explanation:** **What happened:** dependency timeout escaped. **Why:** tests used ideal mocks. **Fix:** 1) rollback; 2) reproduce with failure; 3) add integration/contract test; 4) run; 5) deploy canary; 6) verify. **Prevention:** failure-mode catalog.
- **Scenario Context:** A green pipeline is not proof of user health.
- **Best Practices:** Link incidents to tests.
### Q19. Load test damages a shared environment.
- **One-liner:** Stop load, contain impact, restore state, and isolate future tests.
- **Analogy:** A bridge drill used a public road.
- **Detailed Explanation:** **What happened:** test traffic exhausted shared database. **Why:** no quotas/runbook. **Fix:** 1) stop generator; 2) protect users; 3) restore capacity; 4) clean data; 5) rerun isolated; 6) review. **Prevention:** limits and approvals.
- **Scenario Context:** Monitor both test and real users.
- **Best Practices:** Production tests require explicit safety.
### Q20. Teams bypass a slow gate.
- **One-liner:** Measure bottleneck, optimize, and preserve risk controls.
- **Analogy:** Add inspection lanes instead of removing inspection.
- **Detailed Explanation:** **What happened:** queue caused overrides. **Why:** serial redundant scans. **Fix:** 1) measure; 2) parallelize; 3) cache safely; 4) risk-tier; 5) audit overrides; 6) monitor. **Prevention:** pipeline SLO.
- **Scenario Context:** Keep critical gates mandatory.
- **Best Practices:** Usability is part of control design.

## Coverage Summary
Test layers, contracts, coverage, flake, performance, security, data, quality gates, resilience, and release verification.
