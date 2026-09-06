# AI for DevOps Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Five-layer depth model: **One-liner · Analogy · Detailed Explanation · Scenario Context · Best Practices**

## How to Use This Guide
Q1–Q6 Junior; Q7–Q12 Mid-Level; Q13–Q16 Senior; Q17–Q20 production scenarios.

## SECTION A: JUNIOR LEVEL
### Q1. What is AIOps?
- **One-liner:** AIOps applies analytics/ML to operational telemetry and workflows.
- **Analogy:** A control room with pattern recognition.
- **Detailed Explanation:** It correlates events, detects anomalies, and assists triage; it does not replace evidence.
- **Scenario Context:** Similar alerts are grouped.
- **Best Practices:** Keep humans accountable.
### Q2. What is an LLM?
- **One-liner:** A language model predicts tokens from context.
- **Analogy:** An autocomplete trained on broad examples.
- **Detailed Explanation:** Prompts influence output; generation is probabilistic and may hallucinate.
- **Scenario Context:** It drafts a runbook command requiring review.
- **Best Practices:** Never trust generated commands blindly.
### Q3. What is prompt injection?
- **One-liner:** Untrusted content manipulates an AI system’s instructions.
- **Analogy:** A document slipping fake orders to an assistant.
- **Detailed Explanation:** Retrieved tickets/logs may contain adversarial text; separate data from instructions.
- **Scenario Context:** A log tells an agent to exfiltrate secrets.
- **Best Practices:** Least privilege and output validation.
### Q4. What is anomaly detection?
- **One-liner:** It identifies behavior deviating from a learned or defined baseline.
- **Analogy:** A thermostat noticing an unusual temperature.
- **Detailed Explanation:** Seasonality, drift, false positives, and feature quality matter.
- **Scenario Context:** Traffic anomaly precedes an outage.
- **Best Practices:** Alert only with action and context.
### Q5. What is retrieval-augmented generation?
- **One-liner:** RAG retrieves trusted documents to ground model responses.
- **Analogy:** An assistant consulting a current manual.
- **Detailed Explanation:** Chunking, embeddings, retrieval, citations, and access controls form the path.
- **Scenario Context:** Runbook answers cite current service ownership.
- **Best Practices:** Enforce document permissions.
### Q6. What is human-in-the-loop automation?
- **One-liner:** People approve or supervise consequential AI actions.
- **Analogy:** Autopilot with a pilot.
- **Detailed Explanation:** Define confidence thresholds, previews, approvals, and rollback.
- **Scenario Context:** AI proposes but does not execute firewall changes.
- **Best Practices:** Keep an audit trail.

## SECTION B: MID-LEVEL
### Q7. How can AI assist incident triage?
- **One-liner:** Summarize correlated signals and suggest hypotheses with citations.
- **Analogy:** A junior analyst assembling evidence.
- **Detailed Explanation:** Feed scoped metrics/logs/traces; require links and uncertainty.
- **Scenario Context:** Similar past incidents are surfaced.
- **Best Practices:** Do not let summaries replace raw evidence.
### Q8. What is log summarization risk?
- **One-liner:** Compression can omit rare but decisive evidence.
- **Analogy:** A news summary leaving out the one critical sentence.
- **Detailed Explanation:** Sampling, context windows, privacy, and hallucination affect output.
- **Scenario Context:** A summary misses a security event.
- **Best Practices:** Retain searchable originals.
### Q9. AI for capacity planning?
- **One-liner:** Models forecast demand but require quality history and uncertainty bounds.
- **Analogy:** Weather forecast, not guarantee.
- **Detailed Explanation:** Seasonality, promotions, regressions, and confidence intervals inform scaling.
- **Scenario Context:** Forecast adds capacity before a campaign.
- **Best Practices:** Compare forecast to baseline.
### Q10. AI code generation security?
- **One-liner:** Generated code needs normal review, testing, scanning, and provenance.
- **Analogy:** A fast apprentice whose work still needs inspection.
- **Detailed Explanation:** Models may reproduce vulnerable patterns or licenses; isolate generated changes.
- **Scenario Context:** AI suggests unsafe shell interpolation.
- **Best Practices:** Never bypass review.
### Q11. What is model drift?
- **One-liner:** Performance changes when data or systems change.
- **Analogy:** A map becoming outdated.
- **Detailed Explanation:** Monitor input distributions, precision/recall, feedback, and version changes.
- **Scenario Context:** New log format breaks classifier.
- **Best Practices:** Version and rollback models.
### Q12. How do you evaluate an AI ops assistant?
- **One-liner:** Measure correctness, usefulness, safety, latency, cost, and escalation quality.
- **Analogy:** Evaluate a consultant on advice and consequences.
- **Detailed Explanation:** Golden incidents, adversarial tests, citations, and human ratings create evidence.
- **Scenario Context:** A confident wrong remediation fails evaluation.
- **Best Practices:** Test before production.

## SECTION C: SENIOR LEVEL
### Q13. Design a safe AI operations architecture.
- **One-liner:** Isolate data, ground responses, constrain tools, require approval, and audit actions.
- **Analogy:** A guarded control room assistant.
- **Detailed Explanation:** RAG ACLs, tool allowlists, sandboxing, policy checks, redaction, and immutable logs.
- **Scenario Context:** AI can query metrics but cannot delete resources.
- **Best Practices:** Separate suggestion from execution.
### Q14. How do you protect operational data?
- **One-liner:** Classify, minimize, redact, encrypt, and govern data sent to models.
- **Analogy:** Share only necessary pages from a confidential file.
- **Detailed Explanation:** Tenant isolation, retention, provider contracts, PII filtering, and access logging matter.
- **Scenario Context:** Incident logs contain tokens and customer data.
- **Best Practices:** Prefer private deployment for sensitive workloads.
### Q15. Explain AI agent tool safety.
- **One-liner:** Tools need scoped permissions, validation, rate limits, and human approval.
- **Analogy:** A robot with keys to only one room.
- **Detailed Explanation:** Validate parameters, dry-run, idempotence, allowlists, and circuit breakers.
- **Scenario Context:** Remediation agent cannot run arbitrary shell.
- **Best Practices:** Default deny.
### Q16. How do you govern AI adoption?
- **One-liner:** Define ownership, risk tiers, evaluation, privacy, audit, and incident response.
- **Analogy:** Aviation rules for new aircraft.
- **Detailed Explanation:** Inventory use cases/models, approve data, track versions, and monitor outcomes/cost.
- **Scenario Context:** Production auto-remediation gets stronger controls than summaries.
- **Best Practices:** Document accepted risk.

## SECTION D: PRODUCTION SCENARIOS
### Q17. AI suggests a dangerous remediation.
- **One-liner:** Do not execute, contain incident manually, and improve tool constraints/evaluation.
- **Analogy:** Reject an apprentice’s unsafe repair.
- **Detailed Explanation:** **What happened:** model proposed deleting a database. **Why:** incomplete context/prompt injection. **Fix:** 1) block action; 2) verify evidence; 3) recover manually; 4) add allowlist/approval; 5) replay test; 6) monitor. **Prevention:** dry-run and least privilege.
- **Scenario Context:** Preserve prompt and retrieved documents.
- **Best Practices:** Human approval for destructive actions.
### Q18. RAG returns a stale runbook.
- **One-liner:** Stop recommendation, refresh source/index, and require document version citations.
- **Analogy:** Consultant used an obsolete map.
- **Detailed Explanation:** **What happened:** old failover steps were suggested. **Why:** index retention/access lag. **Fix:** 1) warn responders; 2) use current runbook; 3) rebuild index; 4) verify ACL/version; 5) test retrieval; 6) alert stale docs. **Prevention:** freshness metadata.
- **Scenario Context:** Raw authoritative docs remain primary.
- **Best Practices:** Cite source and revision.
### Q19. AI telemetry cost spikes.
- **One-liner:** Bound input/output volume, sampling, model size, and retention.
- **Analogy:** A consultant billing every photocopy.
- **Detailed Explanation:** **What happened:** every log sent to expensive model. **Why:** no sampling/token budget. **Fix:** 1) throttle; 2) preserve critical evidence; 3) filter/redact; 4) route simple tasks to small model; 5) set budgets; 6) review. **Prevention:** cost SLO.
- **Scenario Context:** Cost controls must not discard audit data.
- **Best Practices:** Measure value per incident.
### Q20. Model behavior changes after upgrade.
- **One-liner:** Pin/version models, evaluate, canary, and rollback outputs safely.
- **Analogy:** A new analyst uses different judgment.
- **Detailed Explanation:** **What happened:** summaries omitted key context. **Why:** model/prompt change. **Fix:** 1) freeze rollout; 2) compare golden incidents; 3) revert; 4) update prompt/eval; 5) canary; 6) monitor. **Prevention:** regression suite and versioned prompts.
- **Scenario Context:** Record model/version and retrieval context.
- **Best Practices:** Treat model changes as releases.

## Coverage Summary
AIOps, LLMs, RAG, prompt injection, privacy, agent safety, evaluation, governance, cost, drift, and incidents.
