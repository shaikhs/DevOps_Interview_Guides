# Terraform Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Five-layer depth model: **One-liner · Analogy · Detailed Explanation · Scenario Context · Best Practices**

## How to Use This Guide
- Q1–Q6 Junior foundations; Q7–Q12 Mid-Level delivery; Q13–Q16 Senior design; Q17–Q20 production scenarios.

## SECTION A: JUNIOR LEVEL

### Q1. What is Terraform?
- **One-liner:** Terraform declaratively provisions infrastructure through providers.
- **Analogy:** A building blueprint reconciled by a construction crew.
- **Detailed Explanation:** Configuration describes resources; `terraform init`, `plan`, and `apply` install providers, preview, and enact changes.
- **Scenario Context:** The same VPC can be reproduced in dev and production from reviewed code.
- **Best Practices:** Pin versions and review plans.
### Q2. What is Terraform state?
- **One-liner:** State maps configuration addresses to real resource identities and attributes.
- **Analogy:** An inventory ledger linking blueprint rooms to actual rooms.
- **Detailed Explanation:** `terraform show` inspects it; state may contain secrets, so encrypt and restrict access.
- **Scenario Context:** Lost state makes Terraform want to recreate existing infrastructure.
- **Best Practices:** Remote state, locking, backups, and no manual edits.
### Q3. What are providers and resources?
- **One-liner:** Providers implement APIs; resources represent managed objects.
- **Analogy:** A translator and the thing being ordered.
- **Detailed Explanation:** `required_providers` pins plugins; `resource "aws_s3_bucket" "logs" {}` declares an object.
- **Scenario Context:** A provider upgrade can change defaults or schemas.
- **Best Practices:** Pin and test provider upgrades.
### Q4. What are variables and outputs?
- **One-liner:** Variables parameterize modules; outputs expose useful results.
- **Analogy:** Inputs on a form and the receipt returned afterward.
- **Detailed Explanation:** Define `variable "region"`; pass `-var-file=prod.tfvars`; expose `output "vpc_id"`.
- **Scenario Context:** CI supplies environment-specific values without copying configuration.
- **Best Practices:** Validate types and mark sensitive outputs.
### Q5. What is a module?
- **One-liner:** A module is a reusable package of Terraform configuration.
- **Analogy:** A standardized appliance installed repeatedly.
- **Detailed Explanation:** A root module calls child modules with inputs; modules should have clear contracts and outputs.
- **Scenario Context:** One network module creates consistent VPCs for many accounts.
- **Best Practices:** Version modules and document compatibility.
### Q6. What is the Terraform lifecycle?
- **One-liner:** Lifecycle meta-arguments control replacement, destruction, and dependency behavior.
- **Analogy:** Rules saying never demolish an occupied building before its replacement exists.
- **Detailed Explanation:** `create_before_destroy`, `prevent_destroy`, and `ignore_changes` alter reconciliation; use sparingly.
- **Scenario Context:** A database replacement is blocked by `prevent_destroy` for safety.
- **Best Practices:** Treat lifecycle exceptions as reviewed risk.

## SECTION B: MID-LEVEL
### Q7. How does dependency ordering work?
- **One-liner:** Terraform builds a dependency graph from references and explicit `depends_on`.
- **Analogy:** A schedule that knows foundations precede walls.
- **Detailed Explanation:** Referencing `aws_vpc.main.id` creates an implicit edge; `depends_on` handles hidden dependencies.
- **Scenario Context:** A policy must wait for a log bucket to exist.
- **Best Practices:** Prefer references over broad `depends_on`.
### Q8. How do workspaces differ from separate state?
- **One-liner:** Workspaces select state instances; separate backends/accounts give stronger isolation.
- **Analogy:** Workspaces are folders; separate accounts are separate buildings.
- **Detailed Explanation:** `terraform workspace select staging` changes state, not credentials or architecture.
- **Scenario Context:** A production apply accidentally targets the wrong workspace.
- **Best Practices:** Use separate states/accounts for production and explicit CI credentials.
### Q9. How do you import existing resources?
- **One-liner:** Import attaches an existing object to a Terraform address; configuration must then match it.
- **Analogy:** Add an existing asset to the inventory before managing it.
- **Detailed Explanation:** `terraform import aws_instance.web i-123`; run plan and add all required arguments.
- **Scenario Context:** A manually created load balancer is brought under control without replacement.
- **Best Practices:** Review drift and import in a controlled branch.
### Q10. How do you handle secrets?
- **One-liner:** Keep secret values outside code/state when possible and minimize exposure.
- **Analogy:** Give a contractor a time-limited vault key, not the master key in the blueprint.
- **Detailed Explanation:** Use a secret manager data source, sensitive variables, encrypted backend, and workload identity; sensitive does not mean absent from state.
- **Scenario Context:** A database password leaked through a committed tfvars file.
- **Best Practices:** Rotate, scan, and use dynamic credentials.
### Q11. What is drift?
- **One-liner:** Drift is a real-world change not represented in configuration or state.
- **Analogy:** Someone remodeled a room without updating the blueprint.
- **Detailed Explanation:** `terraform plan -refresh-only` reveals changes; decide whether to codify or revert them.
- **Scenario Context:** A console security-group edit disappears on the next apply.
- **Best Practices:** Restrict console writes and run scheduled drift checks.
### Q12. How do you test Terraform?
- **One-liner:** Test syntax, plans, policy, modules, and real integration behavior.
- **Analogy:** Check the blueprint, building code, and a model apartment.
- **Detailed Explanation:** Use `terraform fmt -check`, `validate`, plan tests, tflint, Checkov/OPA, and ephemeral applies.
- **Scenario Context:** CI blocks public S3 buckets before deployment.
- **Best Practices:** Test destructive changes and provider upgrades explicitly.

## SECTION C: SENIOR LEVEL
### Q13. Design a multi-team Terraform operating model.
- **One-liner:** Use versioned modules, isolated state, ownership boundaries, and policy gates.
- **Analogy:** A city standardizes utilities while teams own individual buildings.
- **Detailed Explanation:** Separate platform and workload stacks; publish modules; use remote locking, role-based CI, and policy-as-code.
- **Scenario Context:** Teams can deploy applications without changing shared networking.
- **Best Practices:** Define contracts and escalation for breaking changes.
### Q14. How do you manage state at scale?
- **One-liner:** Partition state by lifecycle/blast radius and secure a durable locked backend.
- **Analogy:** Many small ledgers are safer than one ledger for every department.
- **Detailed Explanation:** Use one state per environment/component, encrypted object storage, locking, retention, and recovery drills.
- **Scenario Context:** A network change cannot lock or corrupt every application stack.
- **Best Practices:** Avoid giant states and automate backend bootstrap.
### Q15. How do you handle zero-downtime replacements?
- **One-liner:** Model immutable replacement, health checks, dependencies, and traffic cutover.
- **Analogy:** Build a new bridge beside the old one, test it, then redirect traffic.
- **Detailed Explanation:** Combine `create_before_destroy`, launch templates, target health, and explicit dependency boundaries; confirm names allow parallel existence.
- **Scenario Context:** An AMI update rolls through an ASG without taking the service offline.
- **Best Practices:** Canary and verify capacity before destroying old resources.
### Q16. What belongs in a Terraform review?
- **One-liner:** Review security, blast radius, cost, replacement behavior, and rollback—not just syntax.
- **Analogy:** Approving a construction change by checking exits, utilities, and demolition risk.
- **Detailed Explanation:** Inspect plan additions/deletions, IAM, networking, encryption, tags, timeouts, and provider changes.
- **Scenario Context:** A harmless-looking CIDR edit would replace a production subnet.
- **Best Practices:** Require plan artifacts and two-person approval for critical stacks.

## SECTION D: PRODUCTION SCENARIOS
### Q17. An apply wants to destroy a production database.
- **One-liner:** Stop immediately, determine the graph/config drift, and restore the last safe state.
- **Analogy:** Halt demolition when the building is still occupied.
- **Detailed Explanation:** **What happened:** an identifier change forces replacement. **Why:** immutable attributes changed or state is wrong. **Fix:** 1) do not apply; 2) save plan; 3) inspect `terraform state show`; 4) compare history; 5) restore address/import or migrate safely; 6) plan with `prevent_destroy`. **Prevention:** plan gates and deletion review.
- **Scenario Context:** Confirm backups and ownership before any state surgery.
- **Best Practices:** Never approve destructive plans from an untrusted workspace.
### Q18. A remote state lock is stuck.
- **One-liner:** Verify no apply is running, then remove only a confirmed stale lock.
- **Analogy:** Do not cut a lockbox while someone is inside.
- **Detailed Explanation:** **What happened:** CI crashed holding the lock. **Why:** lock cleanup is not automatic. **Fix:** 1) inspect CI/job; 2) wait or terminate active run; 3) back up state; 4) use documented `force-unlock <ID>`; 5) run refresh-only plan. **Prevention:** backend durability and job timeouts.
- **Scenario Context:** A mistaken force-unlock can cause concurrent writes and state corruption.
- **Best Practices:** Restrict unlock permission and record the incident.
### Q19. A provider upgrade changes resources unexpectedly.
- **One-liner:** Pin, reproduce, review changelog, and roll forward or back deliberately.
- **Analogy:** A building-code update changed how the inspector interprets a plan.
- **Detailed Explanation:** **What happened:** plan shows replacements after upgrade. **Why:** schema/default behavior changed. **Fix:** 1) stop rollout; 2) compare lock file and provider docs; 3) test in clone; 4) pin prior version or adapt config; 5) canary apply. **Prevention:** dependency update automation and compatibility tests.
- **Scenario Context:** Never resolve by deleting state to silence a diff.
- **Best Practices:** Commit `.terraform.lock.hcl` and maintain upgrade notes.
### Q20. Manual emergency changes created drift during an outage.
- **One-liner:** Stabilize first, then codify or revert emergency changes with an auditable plan.
- **Analogy:** Update the blueprint after a temporary repair, not while the roof is collapsing.
- **Detailed Explanation:** **What happened:** console routing fixed traffic but next apply would undo it. **Why:** urgent change bypassed IaC. **Fix:** 1) record exact change; 2) capture current state; 3) test equivalent code; 4) commit and plan; 5) reconcile; 6) review incident. **Prevention:** break-glass workflow and drift alerts.
- **Scenario Context:** Preserve the emergency workaround until the permanent fix is verified.
- **Best Practices:** Make exceptions time-bound and owner-labelled.

## Coverage Summary
- Declarative provisioning, state, modules, dependencies, secrets, drift, testing, scale, safety, and incident recovery.
