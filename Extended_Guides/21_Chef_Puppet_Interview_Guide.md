# Chef & Puppet Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Five-layer depth model: **One-liner · Analogy · Detailed Explanation · Scenario Context · Best Practices**

## How to Use This Guide
Q1–Q6 Junior; Q7–Q12 Mid-Level; Q13–Q16 Senior; Q17–Q20 production scenarios.

## SECTION A: JUNIOR LEVEL
### Q1. What is configuration management?
- **One-liner:** It automates consistent desired configuration across hosts.
- **Analogy:** A fleet mechanic using one specification.
- **Detailed Explanation:** Chef recipes/resources and Puppet manifests/resources declare packages, files, and services.
- **Scenario Context:** Every web server receives the same hardening.
- **Best Practices:** Version code and converge safely.
### Q2. Chef versus Puppet?
- **One-liner:** Chef uses Ruby-based recipes/cookbooks; Puppet uses a declarative DSL/manifests/modules.
- **Analogy:** Two dialects describing the same building standard.
- **Detailed Explanation:** Both use agents/catalogs/resources and support idempotence, but workflows and ecosystems differ.
- **Scenario Context:** Teams choose based on skills and existing estate.
- **Best Practices:** Standardize one approach per domain.
### Q3. What is idempotence?
- **One-liner:** Repeated convergence produces no unnecessary changes.
- **Analogy:** Set a thermostat to a value repeatedly.
- **Detailed Explanation:** `package { 'nginx': ensure => installed }` and Chef `package 'nginx'` express state.
- **Scenario Context:** Agent reruns after a transient failure.
- **Best Practices:** Test second runs.
### Q4. What is a resource?
- **One-liner:** A resource describes one managed system state.
- **Analogy:** A labeled work order.
- **Detailed Explanation:** Package, file, service, user, exec, and template resources have providers/types and properties.
- **Scenario Context:** A changed template notifies a service restart.
- **Best Practices:** Prefer native resources over shell.
### Q5. What is an agent?
- **One-liner:** An agent periodically retrieves policy and converges a node.
- **Analogy:** A scheduled inspector and repairer.
- **Detailed Explanation:** Puppet agent contacts server; Chef Infra Client retrieves cookbooks/policy and runs a converge.
- **Scenario Context:** Drift is corrected automatically.
- **Best Practices:** Control intervals and maintenance windows.
### Q6. What are templates?
- **One-liner:** Templates render configuration from variables.
- **Analogy:** A form filled for each site.
- **Detailed Explanation:** ERB (Chef) and EPP/ERB (Puppet) produce host-specific files.
- **Scenario Context:** Backend address differs by environment.
- **Best Practices:** Validate rendered config before restart.

## SECTION B: MID-LEVEL
### Q7. What is a Chef cookbook?
- **One-liner:** A cookbook packages recipes, attributes, templates, files, resources, and metadata.
- **Analogy:** A reusable repair manual.
- **Detailed Explanation:** Policyfiles/roles/environments select versions and dependencies.
- **Scenario Context:** A base cookbook hardens Linux.
- **Best Practices:** Semantic versioning and tests.
### Q8. What is a Puppet module?
- **One-liner:** A module packages manifests, files, templates, facts, and metadata.
- **Analogy:** A standardized installation kit.
- **Detailed Explanation:** Classes expose parameters; Hiera supplies data; Forge modules add tested functionality.
- **Scenario Context:** One module supports multiple OS families.
- **Best Practices:** Keep data separate from logic.
### Q9. What is Hiera/data binding?
- **One-liner:** It separates environment/host data from Puppet code.
- **Analogy:** Same recipe with local ingredients.
- **Detailed Explanation:** Hierarchy resolves values by node, role, and environment; missing/ambiguous data causes surprises.
- **Scenario Context:** Production endpoint differs from staging.
- **Best Practices:** Validate schemas and precedence.
### Q10. What is Chef Policyfile?
- **One-liner:** Policyfile locks cookbook dependencies and deployable policy revisions.
- **Analogy:** A sealed bill of materials for configuration.
- **Detailed Explanation:** `chef install` resolves; `chef push` publishes a policy group.
- **Scenario Context:** Nodes converge the same cookbook graph.
- **Best Practices:** Commit lockfiles and promote policies.
### Q11. How are notifications used?
- **One-liner:** A changed resource triggers dependent action such as restart.
- **Analogy:** Call a specialist only after the repair.
- **Detailed Explanation:** Chef `notifies`; Puppet `notify/subscribe`; immediate versus delayed timing affects safety.
- **Scenario Context:** One config change causes one restart.
- **Best Practices:** Avoid restart storms.
### Q12. How do you test code?
- **One-liner:** Lint, unit, converge, verify, and integration-test automation.
- **Analogy:** Lab-test a repair kit.
- **Detailed Explanation:** Cookstyle/ChefSpec/Test Kitchen and puppet-lint/rspec-puppet/Beaker cover layers.
- **Scenario Context:** Test Ubuntu and RHEL differences.
- **Best Practices:** Test idempotence and failure paths.

## SECTION C: SENIOR LEVEL
### Q13. Design a fleet configuration platform.
- **One-liner:** Use versioned policy, staged rollout, RBAC, secrets, evidence, and ownership.
- **Analogy:** Regional maintenance depots with one standard.
- **Detailed Explanation:** Controllers/servers, code review, environment promotion, facts, reporting, and rollback form the platform.
- **Scenario Context:** A policy rolls to 5% before fleet-wide convergence.
- **Best Practices:** Measure drift and convergence health.
### Q14. How do you manage secrets?
- **One-liner:** Retrieve secrets at runtime through a vault and avoid repository/state exposure.
- **Analogy:** Temporary keys from a guarded locker.
- **Detailed Explanation:** Chef Vault, encrypted data bags, Puppet Hiera-eyaml, or external secret providers have different trust models.
- **Scenario Context:** A node receives only its database credential.
- **Best Practices:** Rotate and redact reports.
### Q15. How do you handle immutable infrastructure?
- **One-liner:** Bake images for stable base state and use configuration management for bootstrap/legacy convergence.
- **Analogy:** Replace standardized appliances rather than repair every part live.
- **Detailed Explanation:** Packer plus Chef/Puppet reduces drift; agents remain useful for controlled exceptions.
- **Scenario Context:** Rebuilding is safer than modifying thousands of hosts.
- **Best Practices:** Define ownership boundaries.
### Q16. How do you govern modules/cookbooks?
- **One-liner:** Enforce quality, compatibility, security, ownership, and lifecycle policy.
- **Analogy:** Certified spare parts.
- **Detailed Explanation:** Review metadata, dependencies, supported OS, static analysis, tests, signing, and deprecation.
- **Scenario Context:** A module update would change service defaults.
- **Best Practices:** Changelogs and staged promotion.

## SECTION D: PRODUCTION SCENARIOS
### Q17. A cookbook/manifest restarts every service.
- **One-liner:** Stop convergence, identify perpetual change, restore stability, and fix notification logic.
- **Analogy:** A mechanic keeps restarting the engine after every inspection.
- **Detailed Explanation:** **What happened:** template always changed due to timestamp/order. **Why:** non-deterministic rendering. **Fix:** 1) halt rollout; 2) compare diffs; 3) restore config; 4) stabilize template; 5) canary; 6) run twice. **Prevention:** idempotence tests.
- **Scenario Context:** Preserve one node for diagnosis.
- **Best Practices:** Notify only on real change.
### Q18. Fleet drift is not corrected.
- **One-liner:** Check agent connectivity, policy version, permissions, and report freshness.
- **Analogy:** Inspectors stopped receiving work orders.
- **Detailed Explanation:** **What happened:** hosts remain noncompliant. **Why:** certificate/agent/server or catalog failure. **Fix:** 1) inspect reports/logs; 2) restore trust/connectivity; 3) run one node; 4) verify converge; 5) roll out; 6) alert. **Prevention:** agent health monitoring.
- **Scenario Context:** A successful agent process may still fail catalog retrieval.
- **Best Practices:** Alert on stale reports.
### Q19. A module upgrade changes firewall rules.
- **One-liner:** Stop promotion, compare compiled policy, restore safe rules, and stage compatibility.
- **Analogy:** A new building blueprint moved every gate.
- **Detailed Explanation:** **What happened:** SSH access was blocked. **Why:** default changed. **Fix:** 1) use console access; 2) restore last policy; 3) diff generated rules; 4) add explicit parameter; 5) canary; 6) verify. **Prevention:** policy tests and break-glass access.
- **Scenario Context:** Test management path first.
- **Best Practices:** Review generated output, not only source.
### Q20. Secrets appear in a report.
- **One-liner:** Rotate exposed values, restrict reports, and fix redaction/data binding.
- **Analogy:** A vault ledger printed its keys.
- **Detailed Explanation:** **What happened:** debug/resource output contained credential. **Why:** secret marked ordinary data. **Fix:** 1) revoke/rotate; 2) restrict artifacts; 3) remove logs; 4) configure redaction; 5) test; 6) audit. **Prevention:** secret scanning and least privilege.
- **Scenario Context:** Assume retained reports were accessible.
- **Best Practices:** Prefer external runtime retrieval.

## Coverage Summary
Chef/Puppet models, resources, agents, templates, policy/data, secrets, testing, governance, drift, rollout, and incidents.
