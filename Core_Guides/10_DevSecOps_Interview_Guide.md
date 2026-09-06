# DevSecOps Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Five-layer depth model: **One-liner · Analogy · Detailed Explanation · Scenario Context · Best Practices**

## How to Use This Guide
Q1–Q6 Junior; Q7–Q12 Mid-Level; Q13–Q16 Senior; Q17–Q20 production scenarios.

## SECTION A: JUNIOR LEVEL
### Q1. What is DevSecOps?
- **One-liner:** DevSecOps integrates security throughout delivery rather than at the end.
- **Analogy:** Safety checks on every factory station.
- **Detailed Explanation:** Developers, operators, and security share ownership with automated scanning and threat modeling.
- **Scenario Context:** A dependency issue is fixed before release.
- **Best Practices:** Automate feedback early without making security a silo.
### Q2. Authentication versus authorization?
- **One-liner:** Authentication proves identity; authorization grants allowed actions.
- **Analogy:** Showing ID versus knowing which rooms the badge opens.
- **Detailed Explanation:** Use OIDC/OAuth, RBAC, claims, expiry, and audit events.
- **Scenario Context:** A valid token can still lack deployment permission.
- **Best Practices:** Least privilege and short-lived credentials.
### Q3. What is least privilege?
- **One-liner:** Grant only the access required for a task, for only as long as needed.
- **Analogy:** A hotel key opens one room for one stay.
- **Detailed Explanation:** Scope IAM actions/resources, service accounts, network paths, and sudo rules.
- **Scenario Context:** A compromised CI job cannot read production databases.
- **Best Practices:** Review and remove unused access.
### Q4. Why encrypt data?
- **One-liner:** Encryption protects confidentiality in transit and at rest.
- **Analogy:** Sealed letters and locked filing cabinets.
- **Detailed Explanation:** TLS protects transport; KMS-managed keys protect storage; key policy and rotation matter.
- **Scenario Context:** A stolen disk yields ciphertext.
- **Best Practices:** Manage keys separately and test recovery.
### Q5. What is vulnerability scanning?
- **One-liner:** Scanning identifies known weaknesses in code, dependencies, images, and infrastructure.
- **Analogy:** Inspecting ingredients and tools before cooking.
- **Detailed Explanation:** SAST, SCA, container, IaC, and DAST have different coverage and false positives.
- **Scenario Context:** A CVE in a transitive package blocks promotion.
- **Best Practices:** Prioritize exploitability and ownership.
### Q6. What is secret management?
- **One-liner:** Secret management stores, delivers, rotates, and audits sensitive values.
- **Analogy:** A vault issuing temporary keys.
- **Detailed Explanation:** Use Vault/KMS/cloud secret stores, runtime injection, and redaction; base64 is not protection.
- **Scenario Context:** A database password never enters Git or image layers.
- **Best Practices:** Short-lived identity and automatic rotation.

## SECTION B: MID-LEVEL
### Q7. What is SAST versus DAST?
- **One-liner:** SAST analyzes source; DAST tests a running application.
- **Analogy:** Reviewing a recipe versus inspecting the served meal.
- **Detailed Explanation:** SAST catches coding patterns; DAST observes runtime behavior and needs safe test environments.
- **Scenario Context:** A reflected XSS is found only by runtime testing.
- **Best Practices:** Combine tools and triage findings.
### Q8. What is SBOM?
- **One-liner:** An SBOM lists software components and versions in an artifact.
- **Analogy:** A parts list for a machine.
- **Detailed Explanation:** SPDX/CycloneDX enables CVE and license impact analysis; generate at build and attest to digest.
- **Scenario Context:** Locate vulnerable Log4j across fleets quickly.
- **Best Practices:** Keep SBOMs immutable and searchable.
### Q9. What is threat modeling?
- **One-liner:** Threat modeling identifies assets, trust boundaries, threats, and mitigations before implementation.
- **Analogy:** Plan how a building could be entered before choosing locks.
- **Detailed Explanation:** Use STRIDE/data-flow diagrams; prioritize realistic abuse paths and verify controls.
- **Scenario Context:** A public webhook gets signature verification and rate limiting.
- **Best Practices:** Revisit after architecture changes.
### Q10. What is policy as code?
- **One-liner:** Machine-enforced policy makes security requirements repeatable and reviewable.
- **Analogy:** Building codes automatically checked against blueprints.
- **Detailed Explanation:** OPA/Rego, Kyverno, Sentinel, and CI rules can reject public storage or privileged Pods.
- **Scenario Context:** A pull request fails before unsafe infrastructure is applied.
- **Best Practices:** Version policies, explain violations, and support exceptions with expiry.
### Q11. How do you prioritize vulnerabilities?
- **One-liner:** Combine severity, exploitability, exposure, asset criticality, and compensating controls.
- **Analogy:** Triage patients by danger and treatability, not arrival order.
- **Detailed Explanation:** CVSS is input, not a decision; use KEV/intelligence and runtime context.
- **Scenario Context:** An internet-facing exploitable CVE outranks a high score in an isolated tool.
- **Best Practices:** Define SLAs and exception owners.
### Q12. What is secure CI/CD?
- **One-liner:** Protect source, runners, dependencies, artifacts, credentials, and deployment permissions.
- **Analogy:** Secure every checkpoint from warehouse to customer.
- **Detailed Explanation:** Branch protection, ephemeral runners, OIDC, signing, provenance, scanning, and admission controls form a chain.
- **Scenario Context:** A tampered artifact cannot pass signature verification.
- **Best Practices:** Separate build and release trust domains.

## SECTION C: SENIOR LEVEL
### Q13. Design a software supply-chain defense.
- **One-liner:** Establish trusted source, isolated builds, provenance, signatures, scanning, and verified deployment.
- **Analogy:** Tamper-evident manufacturing with a chain-of-custody.
- **Detailed Explanation:** Use SLSA-style provenance, SBOM, reproducible builds, keyless signing, registry policy, and runtime admission.
- **Scenario Context:** A dependency compromise is detected before production.
- **Best Practices:** Protect signing roots and rehearse compromise response.
### Q14. How do you run security without blocking delivery?
- **One-liner:** Automate high-signal controls, risk-tier gates, and fast remediation paths.
- **Analogy:** Airport security uses different lanes for different risk.
- **Detailed Explanation:** Pre-commit feedback, parallel scans, policy thresholds, auto-fix, and governed exceptions reduce friction.
- **Scenario Context:** A low-risk informational finding does not block a critical patch.
- **Best Practices:** Measure remediation time and developer experience.
### Q15. Explain zero trust for DevOps.
- **One-liner:** Never trust network location; continuously verify identity, device, workload, and action.
- **Analogy:** Every room checks the badge, not just the building entrance.
- **Detailed Explanation:** Workload identity, mTLS, segmentation, policy decisions, and telemetry replace broad flat network trust.
- **Scenario Context:** A compromised Pod cannot freely scan the cluster.
- **Best Practices:** Start with high-value paths and avoid brittle over-segmentation.
### Q16. How do you prepare for a cloud breach?
- **One-liner:** Centralize evidence, limit blast radius, automate containment, and rehearse recovery.
- **Analogy:** Fire doors, cameras, and evacuation drills.
- **Detailed Explanation:** Immutable logs, break-glass roles, quarantine controls, key rotation, clean rebuilds, and legal/comms runbooks matter.
- **Scenario Context:** An abused role is disabled while forensics preserves CloudTrail.
- **Best Practices:** Test playbooks and record decision authority.

## SECTION D: PRODUCTION SCENARIOS
### Q17. A critical CVE appears in production.
- **One-liner:** Identify exposure, contain risk, patch/rebuild, verify, and prevent recurrence.
- **Analogy:** Recall an affected batch and inspect its distribution.
- **Detailed Explanation:** **What happened:** scanner found exploitable library. **Why:** stale base image. **Fix:** 1) inventory SBOM; 2) assess exposure; 3) apply mitigation; 4) rebuild/sign/scan; 5) canary rollout; 6) verify. **Prevention:** rebuild SLAs and inventory.
- **Scenario Context:** Prioritize internet-facing assets.
- **Best Practices:** Document any time-bound exception.
### Q18. A production secret leaked in logs.
- **One-liner:** Revoke first, investigate, rotate, remove exposure, and fix logging.
- **Analogy:** Change a lock after its key is photographed.
- **Detailed Explanation:** **What happened:** debug printed token. **Why:** unsafe logging. **Fix:** 1) revoke; 2) audit use; 3) rotate dependents; 4) restrict logs; 5) add redaction tests; 6) notify stakeholders. **Prevention:** secret scanners and no-log controls.
- **Scenario Context:** Assume all readers accessed retained logs.
- **Best Practices:** Prefer ephemeral tokens.
### Q19. A malicious image reaches the registry.
- **One-liner:** Quarantine artifacts, stop admission, investigate provenance, and rebuild trusted outputs.
- **Analogy:** Lock a warehouse after counterfeit goods are found.
- **Detailed Explanation:** **What happened:** unsigned image was pushed. **Why:** registry write was broad. **Fix:** 1) block promotion; 2) revoke credentials; 3) audit pushes; 4) scan/sign clean image; 5) rotate runtime; 6) verify deployments. **Prevention:** immutable tags, signing, RBAC.
- **Scenario Context:** Treat deployed copies as suspect.
- **Best Practices:** Separate build and deploy identities.
### Q20. Security controls create unacceptable pipeline delay.
- **One-liner:** Measure bottlenecks, parallelize safe checks, and risk-tier gates without deleting protection.
- **Analogy:** Open more inspection lanes, do not remove inspections.
- **Detailed Explanation:** **What happened:** teams bypassed scans. **Why:** serial low-value checks. **Fix:** 1) measure stages; 2) cache safely; 3) parallelize; 4) gate only material risk; 5) auto-remediate; 6) monitor bypasses. **Prevention:** security platform SLO.
- **Scenario Context:** Preserve mandatory release evidence.
- **Best Practices:** Security must be usable to be effective.

## Coverage Summary
Identity, secrets, encryption, vulnerability management, supply chain, policy, zero trust, incident response, and practical delivery trade-offs.
