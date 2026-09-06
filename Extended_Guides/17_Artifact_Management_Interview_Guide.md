# Artifact Management Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Five-layer depth model: **One-liner · Analogy · Detailed Explanation · Scenario Context · Best Practices**

## How to Use This Guide
Q1–Q6 Junior; Q7–Q12 Mid-Level; Q13–Q16 Senior; Q17–Q20 production scenarios.

## SECTION A: JUNIOR LEVEL
### Q1. What is an artifact?
- **One-liner:** A versioned build output consumed by deployment.
- **Analogy:** A sealed product from a factory.
- **Detailed Explanation:** Binaries, packages, images, charts, and SBOMs are artifacts.
- **Scenario Context:** Production uses the tested artifact.
- **Best Practices:** Immutable identity and retention.
### Q2. What is a repository?
- **One-liner:** A repository stores and serves artifact versions.
- **Analogy:** A warehouse catalog.
- **Detailed Explanation:** Registries provide auth, metadata, layers, and download APIs.
- **Scenario Context:** CI publishes to a private repository.
- **Best Practices:** Separate read/write roles.
### Q3. Tag versus digest?
- **One-liner:** A tag is mutable naming; a digest is content identity.
- **Analogy:** Nickname versus fingerprint.
- **Detailed Explanation:** Deploy `image@sha256:...` for reproducibility.
- **Scenario Context:** `latest` changes unexpectedly.
- **Best Practices:** Promote by digest.
### Q4. What is checksum verification?
- **One-liner:** A checksum detects content changes.
- **Analogy:** Tamper seal.
- **Detailed Explanation:** SHA-256 identifies bytes; signatures also authenticate the signer.
- **Scenario Context:** A download is rejected after checksum mismatch.
- **Best Practices:** Verify through trusted channels.
### Q5. What is retention?
- **One-liner:** Retention determines how long artifacts remain available.
- **Analogy:** Warehouse storage policy.
- **Detailed Explanation:** Keep release/rollback versions; expire unreferenced intermediates.
- **Scenario Context:** Aggressive cleanup removes rollback image.
- **Best Practices:** Protect production dependencies.
### Q6. What is artifact promotion?
- **One-liner:** Promotion moves a tested artifact between trust environments.
- **Analogy:** Inspected goods entering better warehouses.
- **Detailed Explanation:** Metadata, approvals, signatures, and digest remain unchanged.
- **Scenario Context:** Staging digest becomes production digest.
- **Best Practices:** Never rebuild during promotion.

## SECTION B: MID-LEVEL
### Q7. What is provenance?
- **One-liner:** Provenance records source, builder, inputs, and process.
- **Analogy:** Chain-of-custody paperwork.
- **Detailed Explanation:** Attest commit SHA, workflow, dependencies, and timestamp.
- **Scenario Context:** Investigators trace a binary to a job.
- **Best Practices:** Sign attestations.
### Q8. What is an SBOM?
- **One-liner:** An SBOM lists components in an artifact.
- **Analogy:** Parts list.
- **Detailed Explanation:** SPDX/CycloneDX supports vulnerability and license response.
- **Scenario Context:** Find all builds containing a library.
- **Best Practices:** Generate at build time.
### Q9. How do repositories proxy dependencies?
- **One-liner:** A proxy caches upstream packages under organizational control.
- **Analogy:** Local supplier stocking common parts.
- **Detailed Explanation:** It improves availability and supports allowlists, checksums, and audit.
- **Scenario Context:** Builds continue during upstream outage.
- **Best Practices:** Pin versions and inspect upstream risk.
### Q10. What is artifact signing?
- **One-liner:** Signing binds an artifact to an authenticated signer.
- **Analogy:** A notary seal.
- **Detailed Explanation:** Cosign/Notary signatures are verified before deployment.
- **Scenario Context:** Unsigned image is rejected.
- **Best Practices:** Protect keys or use keyless identity.
### Q11. How do you manage packages?
- **One-liner:** Use version constraints, lockfiles, metadata, and promotion.
- **Analogy:** Approved parts catalog.
- **Detailed Explanation:** npm/pip/Maven/OS repositories resolve dependencies; lockfiles improve repeatability.
- **Scenario Context:** A transitive update changes behavior.
- **Best Practices:** Automated updates with tests.
### Q12. What is repository federation?
- **One-liner:** Federation distributes or mirrors artifacts between repositories/regions.
- **Analogy:** Regional warehouses with synchronized stock.
- **Detailed Explanation:** Replication, consistency, latency, and access policy must be defined.
- **Scenario Context:** A region pulls from its nearest mirror.
- **Best Practices:** Verify digest after replication.

## SECTION C: SENIOR LEVEL
### Q13. Design artifact governance.
- **One-liner:** Standardize naming, provenance, scanning, signing, promotion, retention, and ownership.
- **Analogy:** A regulated supply chain.
- **Detailed Explanation:** Registry policy, SBOM, quarantine, immutable repos, RBAC, and audit create trust.
- **Scenario Context:** Production admits only approved digests.
- **Best Practices:** Make exceptions expiring and visible.
### Q14. How do you guarantee reproducible builds?
- **One-liner:** Pin inputs, isolate builders, control time/environment, and compare outputs.
- **Analogy:** Same recipe, ingredients, oven, and measurements.
- **Detailed Explanation:** Lockfiles, pinned base digests, deterministic timestamps, hermetic builds, and provenance help.
- **Scenario Context:** Rebuilding a release yields same digest or explained difference.
- **Best Practices:** Verify reproducibility continuously.
### Q15. How do you manage artifact lifecycle?
- **One-liner:** Retain what recovery needs and delete safely what it does not.
- **Analogy:** Archive legal records, recycle drafts.
- **Detailed Explanation:** Classify releases, snapshots, caches, legal holds, and rollback windows.
- **Scenario Context:** Disaster recovery requires old images.
- **Best Practices:** Test restore before deletion.
### Q16. How do you respond to a compromised artifact?
- **One-liner:** Quarantine, identify consumers, revoke trust, rebuild, and rotate deployment credentials.
- **Analogy:** Recall counterfeit products.
- **Detailed Explanation:** Audit pushes/pulls, block digest admission, preserve evidence, and redeploy known-good versions.
- **Scenario Context:** Cached copies may remain on nodes.
- **Best Practices:** Exercise the recall playbook.

## SECTION D: PRODUCTION SCENARIOS
### Q17. Registry cleanup deleted rollback artifacts.
- **One-liner:** Restore from replica/backup, halt risky cleanup, and fix retention classification.
- **Analogy:** Warehouse disposed of emergency stock.
- **Detailed Explanation:** **What happened:** rollback image missing. **Why:** policy kept tags, not deployments. **Fix:** 1) stop cleanup; 2) identify digest; 3) restore/copy; 4) validate signature; 5) update retention; 6) test. **Prevention:** protected release set.
- **Scenario Context:** Tags may be moved or deleted.
- **Best Practices:** Retain by digest and dependency graph.
### Q18. Build pulls a malicious dependency.
- **One-liner:** Quarantine output, investigate source, rebuild from trusted proxy, and audit consumers.
- **Analogy:** Recall a part from the supply chain.
- **Detailed Explanation:** **What happened:** package was compromised. **Why:** unpinned upstream resolution. **Fix:** 1) stop promotion; 2) identify builds; 3) revoke; 4) pin/replace; 5) scan/sign; 6) redeploy. **Prevention:** proxy allowlists and lockfiles.
- **Scenario Context:** Check build runners and credentials.
- **Best Practices:** Generate SBOM/provenance.
### Q19. Artifact promotion uses wrong digest.
- **One-liner:** Stop deployment, trace metadata, rollback, and make promotion content-addressed.
- **Analogy:** Correct label attached to wrong box.
- **Detailed Explanation:** **What happened:** mutable tag resolved differently. **Why:** promotion copied names not digest. **Fix:** 1) identify running digest; 2) rollback; 3) compare registry manifests; 4) promote digest; 5) add assertion; 6) verify. **Prevention:** immutable refs.
- **Scenario Context:** Preserve audit records.
- **Best Practices:** Display digest in deployment UI.
### Q20. Artifact storage costs spike.
- **One-liner:** Attribute storage, remove safe duplicates, and optimize retention/replication.
- **Analogy:** Too many warehouses and copies.
- **Detailed Explanation:** **What happened:** layers and build outputs grew. **Why:** no lifecycle policy. **Fix:** 1) analyze usage; 2) protect releases; 3) expire intermediates; 4) deduplicate; 5) tune replicas; 6) alert. **Prevention:** cost budgets.
- **Scenario Context:** Do not delete compliance-held artifacts.
- **Best Practices:** Cost per release and owner.

## Coverage Summary
Artifact identity, repositories, packages, provenance, SBOM, signing, lifecycle, promotion, supply-chain response, and cost.
