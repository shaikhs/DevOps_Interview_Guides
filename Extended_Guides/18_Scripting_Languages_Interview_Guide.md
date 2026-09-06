# Scripting Languages Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Five-layer depth model: **One-liner · Analogy · Detailed Explanation · Scenario Context · Best Practices**

## How to Use This Guide
Q1–Q6 Junior; Q7–Q12 Mid-Level; Q13–Q16 Senior; Q17–Q20 production scenarios.

## SECTION A: JUNIOR LEVEL
### Q1. Bash variables?
- **One-liner:** Variables store shell values; quote expansions deliberately.
- **Analogy:** Labels on boxes.
- **Detailed Explanation:** `name="prod"; echo "$name"`; shell variables are strings.
- **Scenario Context:** Unquoted paths split unexpectedly.
- **Best Practices:** `set -euo pipefail` with understanding.
### Q2. Exit codes?
- **One-liner:** Zero means success by convention; nonzero signals failure.
- **Analogy:** A job completion status.
- **Detailed Explanation:** `$?`, `&&`, `||`, and pipeline status control flow.
- **Scenario Context:** CI proceeds despite a failed command.
- **Best Practices:** Check errors explicitly.
### Q3. Python virtual environments?
- **One-liner:** They isolate Python dependencies per project.
- **Analogy:** Separate toolboxes.
- **Detailed Explanation:** `python -m venv .venv`; lock dependencies and activate in CI.
- **Scenario Context:** Two services need incompatible versions.
- **Best Practices:** Pin and scan packages.
### Q4. Arrays and loops?
- **One-liner:** Collections and loops automate repeated work safely.
- **Analogy:** Process each item in a checklist.
- **Detailed Explanation:** Bash `for x in "${items[@]}"`; Python `for item in items`.
- **Scenario Context:** Batch host checks.
- **Best Practices:** Handle empty/space-containing values.
### Q5. Streams and pipes?
- **One-liner:** stdin/stdout/stderr connect commands and diagnostics.
- **Analogy:** Conveyor belts for data and warnings.
- **Detailed Explanation:** `cmd 2>err | tee out`; avoid parsing fragile human output.
- **Scenario Context:** Errors disappear when stderr is ignored.
- **Best Practices:** Preserve stderr and use structured output.
### Q6. JSON processing?
- **One-liner:** Use a parser such as `jq`, not regex, for JSON.
- **Analogy:** Read a database, not a screenshot.
- **Detailed Explanation:** `jq -r '.items[].name' response.json`.
- **Scenario Context:** API field order changes safely.
- **Best Practices:** Validate schemas and handle missing fields.

## SECTION B: MID-LEVEL
### Q7. Idempotent scripts?
- **One-liner:** Re-running produces the same intended state.
- **Analogy:** Set thermostat, do not blindly turn knob.
- **Detailed Explanation:** Check existence/state before create/delete; return clear status.
- **Scenario Context:** Retry after network failure does not duplicate resources.
- **Best Practices:** Dry-run and test twice.
### Q8. Shell quoting?
- **One-liner:** Quoting prevents unwanted splitting, globbing, and expansion.
- **Analogy:** Package a phrase as one object.
- **Detailed Explanation:** `"$file"` differs from `$file`; use arrays for arguments.
- **Scenario Context:** A filename with spaces breaks deployment.
- **Best Practices:** ShellCheck.
### Q9. Python exceptions?
- **One-liner:** Exceptions represent failures and should be caught at meaningful boundaries.
- **Analogy:** Escalate a failed delivery to a dispatcher.
- **Detailed Explanation:** `try/except`, context, retries, and nonzero exit codes communicate failure.
- **Scenario Context:** API timeout retries then fails clearly.
- **Best Practices:** Do not catch everything silently.
### Q10. Concurrency?
- **One-liner:** Parallel work improves throughput but needs bounded resources and coordination.
- **Analogy:** More checkout lanes require staffing.
- **Detailed Explanation:** Python threads/processes/async; Bash background jobs with `wait`.
- **Scenario Context:** Unbounded scans overload an API.
- **Best Practices:** Limit concurrency and preserve errors.
### Q11. API automation?
- **One-liner:** Authenticate, validate responses, retry safe operations, and log identifiers.
- **Analogy:** A careful clerk using a service window.
- **Detailed Explanation:** Timeouts, pagination, rate limits, idempotency keys, and backoff matter.
- **Scenario Context:** A retry must not duplicate a create request.
- **Best Practices:** Never log tokens.
### Q12. Testing scripts?
- **One-liner:** Unit-test logic and integration-test side effects in disposable environments.
- **Analogy:** Test a tool in a workshop before a live repair.
- **Detailed Explanation:** pytest/Bats, mocks, fixtures, ShellCheck, and CI.
- **Scenario Context:** A destructive branch is tested without production access.
- **Best Practices:** Test failure paths.

## SECTION C: SENIOR LEVEL
### Q13. Design reliable automation.
- **One-liner:** Define inputs, outputs, retries, timeouts, observability, and rollback.
- **Analogy:** A production machine with emergency stops.
- **Detailed Explanation:** Typed config, structured logs, correlation IDs, bounded concurrency, and idempotent actions.
- **Scenario Context:** A job resumes after worker loss.
- **Best Practices:** Prefer platform APIs/IaC for durable state.
### Q14. When Bash versus Python?
- **One-liner:** Bash composes Unix tools; Python suits complex logic, data, and testing.
- **Analogy:** Pocket knife versus workshop tools.
- **Detailed Explanation:** Choose based on error handling, portability, dependencies, and maintainability.
- **Scenario Context:** A 300-line Bash parser becomes a typed Python module.
- **Best Practices:** Keep scripts small and documented.
### Q15. How do you secure automation?
- **One-liner:** Validate inputs, minimize credentials, avoid injection, and audit actions.
- **Analogy:** A tool accepts only safe materials and has limited keys.
- **Detailed Explanation:** Avoid `eval`, parameterize commands, use allowlists, secret managers, and least privilege.
- **Scenario Context:** User input must not become shell code.
- **Best Practices:** Static analysis and adversarial tests.
### Q16. How do you operate scripts at scale?
- **One-liner:** Use a scheduler/queue, concurrency limits, checkpoints, and centralized telemetry.
- **Analogy:** Dispatch fleet with bounded traffic.
- **Detailed Explanation:** Job identity, retries, dead letters, leases, and metrics make execution reliable.
- **Scenario Context:** Failed hosts are retried without repeating successes.
- **Best Practices:** Design for interruption.

## SECTION D: PRODUCTION SCENARIOS
### Q17. A cleanup script deletes wrong files.
- **One-liner:** Stop it, restore from backup, identify unsafe expansion, and add dry-run safeguards.
- **Analogy:** A vacuum with a broken boundary.
- **Detailed Explanation:** **What happened:** empty variable expanded to root path. **Why:** missing validation. **Fix:** 1) kill job; 2) preserve evidence; 3) restore; 4) require nonempty/allowed path; 5) dry-run; 6) canary. **Prevention:** ShellCheck and tests.
- **Scenario Context:** Never “fix” by deleting more.
- **Best Practices:** Explicit allowlists.
### Q18. API script creates duplicates after timeout.
- **One-liner:** Use idempotency keys and reconcile before retrying.
- **Analogy:** Ask whether the order arrived before placing it again.
- **Detailed Explanation:** **What happened:** response was lost after create. **Why:** retry was non-idempotent. **Fix:** 1) query by key; 2) stop duplicates; 3) add idempotency key; 4) backoff; 5) test timeout; 6) audit. **Prevention:** safe retry policy.
- **Scenario Context:** Network timeout does not prove server failure.
- **Best Practices:** Store operation IDs.
### Q19. Python job exhausts memory.
- **One-liner:** Bound input/concurrency, stream data, and restore service.
- **Analogy:** Loading an entire warehouse into one van.
- **Detailed Explanation:** **What happened:** process OOMed. **Why:** read-all and unbounded workers. **Fix:** 1) stop/contain; 2) profile; 3) stream/chunk; 4) cap workers; 5) load test; 6) monitor. **Prevention:** resource limits.
- **Scenario Context:** Keep partial progress checkpointed.
- **Best Practices:** Measure peak memory.
### Q20. A script succeeds but leaves partial state.
- **One-liner:** Add transactions/checkpoints, compensation, and explicit failure reporting.
- **Analogy:** A move completed only halfway.
- **Detailed Explanation:** **What happened:** step 4 failed after steps 1–3. **Why:** no rollback model. **Fix:** 1) mark incomplete; 2) reconcile state; 3) compensate safely; 4) resume idempotently; 5) test faults; 6) alert. **Prevention:** state machine design.
- **Scenario Context:** Report partial success, not exit 0.
- **Best Practices:** Prefer declarative controllers for convergence.

## Coverage Summary
Bash, Python, shell safety, APIs, JSON, concurrency, testing, security, idempotence, and production automation failures.
