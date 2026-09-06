# Monitoring & Observability Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Five-layer depth model: **One-liner · Analogy · Detailed Explanation · Scenario Context · Best Practices**

## How to Use This Guide
Q1–Q6 Junior; Q7–Q12 Mid-Level; Q13–Q16 Senior; Q17–Q20 production scenarios.

## SECTION A: JUNIOR LEVEL
### Q1. Metrics, logs, and traces?
- **One-liner:** Metrics are numeric trends, logs are event records, traces follow requests.
- **Analogy:** Dashboard gauges, diary entries, and a parcel tracking number.
- **Detailed Explanation:** Use counters/gauges/histograms; structured logs carry context; trace spans show latency across services.
- **Scenario Context:** A trace links an API timeout to a database span.
- **Best Practices:** Correlate with request IDs and control cardinality.
### Q2. What is an SLI/SLO/SLA?
- **One-liner:** SLI measures reliability, SLO sets the target, SLA is an external commitment.
- **Analogy:** Speedometer, desired speed, and a delivery promise.
- **Detailed Explanation:** Availability, latency, freshness, and correctness are common SLIs; error budget is `1-SLO`.
- **Scenario Context:** Release freezes follow budget burn.
- **Best Practices:** Measure user-visible outcomes.
### Q3. Counter versus gauge?
- **One-liner:** Counters increase monotonically; gauges move up and down.
- **Analogy:** Odometer versus thermometer.
- **Detailed Explanation:** Rate counters with `rate()`; use gauges for memory or queue depth.
- **Scenario Context:** Counter reset after restart is normal.
- **Best Practices:** Document units and labels.
### Q4. What is an alert?
- **One-liner:** An alert is an actionable signal requiring a response.
- **Analogy:** Smoke alarm, not a dashboard decoration.
- **Detailed Explanation:** Define threshold, duration, severity, owner, runbook, and notification route.
- **Scenario Context:** Page on sustained customer-impacting latency, ticket on capacity trend.
- **Best Practices:** Every page must have an action.
### Q5. Why use structured logging?
- **One-liner:** Machine-readable fields enable reliable search and aggregation.
- **Analogy:** Database rows instead of handwritten notes.
- **Detailed Explanation:** Emit JSON fields such as `timestamp`, `level`, `service`, `trace_id`, and `error`; avoid secrets.
- **Scenario Context:** Query all 500s for one trace.
- **Best Practices:** UTC, schemas, redaction, and retention.
### Q6. What is a dashboard?
- **One-liner:** A dashboard presents related signals for diagnosis and decisions.
- **Analogy:** A vehicle instrument panel.
- **Detailed Explanation:** Include traffic, errors, latency, saturation, dependencies, and deploy markers.
- **Scenario Context:** The on-call sees impact and likely bottleneck in one view.
- **Best Practices:** Prefer focused service dashboards to metric walls.

## SECTION B: MID-LEVEL
### Q7. Explain RED and USE.
- **One-liner:** RED covers request rate, errors, duration; USE covers utilization, saturation, errors.
- **Analogy:** Customer queue health and machine capacity health.
- **Detailed Explanation:** Apply RED to services and USE to resources; combine with business signals.
- **Scenario Context:** High latency with normal CPU points to downstream saturation.
- **Best Practices:** Start with these, then add domain-specific signals.
### Q8. What is histogram quantile accuracy?
- **One-liner:** Histograms estimate distributions and quantiles from buckets; bucket design matters.
- **Analogy:** Counting runners in time bands rather than recording every time.
- **Detailed Explanation:** Prometheus `histogram_quantile()` needs suitable buckets; summaries calculate locally and aggregate poorly.
- **Scenario Context:** A p99 hidden by coarse buckets misleads capacity decisions.
- **Best Practices:** Use latency-appropriate buckets and validate against traces.
### Q9. How do you control cardinality?
- **One-liner:** Limit label combinations because each creates storage and query cost.
- **Analogy:** Do not create a filing cabinet drawer for every visitor.
- **Detailed Explanation:** Never label metrics with unbounded user IDs/request paths; normalize routes and sample traces.
- **Scenario Context:** A deployment causes millions of new time series.
- **Best Practices:** Review labels in code and alert on series growth.
### Q10. How does distributed tracing work?
- **One-liner:** A trace consists of timed spans propagated across service boundaries.
- **Analogy:** A package barcode scanned at every depot.
- **Detailed Explanation:** W3C trace context propagates IDs; instrument HTTP, queues, DB calls; sampling balances cost and visibility.
- **Scenario Context:** Fan-out spans reveal one slow dependency.
- **Best Practices:** Preserve trace IDs in logs and sample errors.
### Q11. What is alert fatigue?
- **One-liner:** Excessive or non-actionable alerts desensitize responders.
- **Analogy:** A fire alarm that rings for burnt toast.
- **Detailed Explanation:** Tune thresholds, duration, grouping, inhibition, and severity; remove duplicate pages.
- **Scenario Context:** A noisy disk warning hides a real outage.
- **Best Practices:** Review pages after incidents and track false positives.
### Q12. How do you monitor Kubernetes?
- **One-liner:** Monitor control plane, nodes, workloads, events, and user outcomes.
- **Analogy:** Monitor both railway signals and passenger arrivals.
- **Detailed Explanation:** Collect kube-state-metrics, cAdvisor/node metrics, logs, events, probes, and API audit logs.
- **Scenario Context:** Pending Pods plus node pressure identifies scheduling failure.
- **Best Practices:** Alert on symptoms and saturation, not every restart.

## SECTION C: SENIOR LEVEL
### Q13. Design an observability platform.
- **One-liner:** Standardize telemetry, collection, storage tiers, access, retention, and ownership.
- **Analogy:** A national infrastructure of sensors and control rooms.
- **Detailed Explanation:** Use OpenTelemetry, regional collectors, durable metrics/logs/traces, sampling, tenant isolation, and query budgets.
- **Scenario Context:** An incident remains diagnosable during a regional service failure.
- **Best Practices:** Treat telemetry as production infrastructure.
### Q14. How do you design SLOs?
- **One-liner:** Choose user journeys, measurable indicators, realistic targets, and error-budget policy.
- **Analogy:** Promise the outcome customers care about, not engine temperature.
- **Detailed Explanation:** Define availability/latency windows, exclusions, burn-rate alerts, and release response.
- **Scenario Context:** Checkout success matters more than raw CPU.
- **Best Practices:** Review targets with product and revise from evidence.
### Q15. Explain burn-rate alerting.
- **One-liner:** Burn rate detects how quickly an error budget is being consumed.
- **Analogy:** A fuel leak warning based on speed, not just tank level.
- **Detailed Explanation:** Combine short and long windows to catch fast outages and slow degradation; route severity by remaining budget.
- **Scenario Context:** A 1-hour burn page catches a release while a 3-day window catches drift.
- **Best Practices:** Test rules against historical incidents.
### Q16. How do you reduce observability cost?
- **One-liner:** Control volume, cardinality, retention, sampling, and query inefficiency without losing critical evidence.
- **Analogy:** Archive every legal document, not every photocopy.
- **Detailed Explanation:** Tier storage, tail-sample errors, aggregate high-volume metrics, and redact early.
- **Scenario Context:** Debug traces are retained longer than routine access logs.
- **Best Practices:** Price telemetry per service and review monthly.

## SECTION D: PRODUCTION SCENARIOS
### Q17. Users report slowness but dashboards are green.
- **One-liner:** Validate telemetry coverage, segment users, and inspect traces/business indicators.
- **Analogy:** The central gauge works while one neighborhood loses water.
- **Detailed Explanation:** **What happened:** aggregate p95 hid a regional/client cohort. **Why:** poor dimensions and sampling. **Fix:** 1) reproduce; 2) segment by region/version; 3) inspect traces/logs; 4) compare dependency latency; 5) remediate; 6) add SLI. **Prevention:** journey-based SLOs and coverage tests.
- **Scenario Context:** Averages are not proof of universal health.
- **Best Practices:** Monitor cohorts without unbounded labels.
### Q18. Alert storm during an outage.
- **One-liner:** Page on the primary symptom, suppress dependent alerts, and fix routing after recovery.
- **Analogy:** One broken power line triggers alarms in every house.
- **Detailed Explanation:** **What happened:** one DNS failure generated thousands of pages. **Why:** no dependency grouping/inhibition. **Fix:** 1) declare incident; 2) mute duplicates; 3) restore DNS; 4) verify services; 5) tune rules; 6) replay test. **Prevention:** topology-aware alerting.
- **Scenario Context:** Preserve alerts/logs for timeline evidence.
- **Best Practices:** Keep one accountable page owner.
### Q19. Logs are missing during an incident.
- **One-liner:** Restore collection safely, use alternate signals, then fix buffering and capacity.
- **Analogy:** The camera recorder filled its disk during the break-in.
- **Detailed Explanation:** **What happened:** collector dropped records. **Why:** backpressure and no disk budget. **Fix:** 1) check agent/queue; 2) use metrics/traces; 3) expand or drain collector; 4) recover buffered logs; 5) validate; 6) tune. **Prevention:** pipeline SLOs and loss alerts.
- **Scenario Context:** Never assume absence of logs means absence of events.
- **Best Practices:** Protect audit logs separately.
### Q20. A false alert caused an unnecessary failover.
- **One-liner:** Stabilize state, reverse safely, identify bad signal, and add approval/guardrails.
- **Analogy:** A faulty smoke alarm evacuated the building into danger.
- **Detailed Explanation:** **What happened:** transient scrape failure triggered automation. **Why:** alert treated missing data as outage. **Fix:** 1) stop automation; 2) assess state; 3) restore correct endpoint; 4) repair rule; 5) test no-data behavior; 6) audit. **Prevention:** multi-signal confirmation.
- **Scenario Context:** Automation should fail closed for destructive actions.
- **Best Practices:** Require health evidence and rate limits.

## Coverage Summary
Telemetry, metrics, logs, traces, SLOs, alert design, cost, platform architecture, and incidents.
