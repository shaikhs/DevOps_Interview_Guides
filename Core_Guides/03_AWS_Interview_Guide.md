# AWS Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Every answer follows the 5-layer depth model: **One-liner · Analogy · Detailed Explanation · Scenario Context · Best Practices**

## How to Use This Guide
- **Q1–Q6 Junior:** AWS foundations; **Q7–Q12 Mid-Level:** production patterns; **Q13–Q16 Senior:** architecture and trade-offs; **Q17–Q20 Scenarios:** incident reasoning.
- Use the detailed layer to demonstrate hands-on skill, and the best-practices layer to explain safe operational decisions.

## SECTION A: JUNIOR LEVEL

### Q1. What is an AWS Region and Availability Zone?
- **One-liner:** A Region is a geographic area; Availability Zones are isolated datacenters within it.
- **Analogy:** A Region is a city and AZs are separate fire stations that can serve the same city.
- **Detailed Explanation:** Regions have independent control planes and multiple AZs connected by low-latency links; deploy stateful or critical workloads across at least two AZs. `aws ec2 describe-availability-zones --region us-east-1` lists them.
- **Scenario Context:** An instance survives an AZ failure when its load balancer and replicas span AZs.
- **Best Practices:** Choose regions for latency, sovereignty, and service availability; never treat AZs as a single failure domain.

### Q2. What is IAM and how do users, roles, and policies differ?
- **One-liner:** IAM controls AWS authentication and authorization through identities and JSON policies.
- **Analogy:** A role is a temporary access badge; a user is a named employee; a policy is the badge’s access list.
- **Detailed Explanation:** Prefer roles and STS temporary credentials. A policy matches `Effect`, `Action`, `Resource`, and optional `Condition`; explicit deny overrides allow. `aws iam simulate-principal-policy` helps test access.
- **Scenario Context:** An EC2 deployment needs S3 access without storing keys on disk, so it assumes an instance profile role.
- **Best Practices:** Apply least privilege, require MFA for humans, deny public access broadly, and review unused permissions.

### Q3. Compare S3, EBS, and EFS.
- **One-liner:** S3 is object storage, EBS is block storage attached to instances, and EFS is shared elastic file storage.
- **Analogy:** S3 is a warehouse catalog, EBS is a hard drive, and EFS is a network file room.
- **Detailed Explanation:** S3 uses buckets/objects and APIs; EBS volumes persist independently of an instance but are AZ-scoped; EFS exposes NFS across AZs. `aws s3 sync build/ s3://site/` uploads objects.
- **Scenario Context:** Backups go to versioned S3, databases use provisioned-IOPS EBS, and shared uploads use EFS.
- **Best Practices:** Encrypt by default, enable S3 versioning/lifecycle, snapshot EBS, and avoid using EFS as a low-latency database disk.

### Q4. What does a VPC contain?
- **One-liner:** A VPC is an isolated network defined by CIDR ranges, subnets, routes, and security controls.
- **Analogy:** It is a campus: subnets are buildings, route tables are roads, and security groups are door guards.
- **Detailed Explanation:** Public subnets route through an Internet Gateway; private subnets use NAT for egress; NACLs are stateless subnet filters and security groups are stateful ENI firewalls. Inspect paths with `aws ec2 describe-route-tables`.
- **Scenario Context:** An application tier stays private while an ALB in public subnets accepts HTTPS.
- **Best Practices:** Plan non-overlapping CIDRs for future peering, keep data private, and log flow decisions with VPC Flow Logs.

### Q5. What are security groups and NACLs?
- **One-liner:** Security groups are stateful instance-level allow rules; NACLs are stateless subnet-level allow/deny rules.
- **Analogy:** A security group is a receptionist who remembers visitors; a NACL is a gate checking every inbound and outbound trip.
- **Detailed Explanation:** Return traffic is automatically allowed by a security group; NACLs require ephemeral return ports. Use source security groups rather than broad IP ranges: `aws ec2 authorize-security-group-ingress --group-id sg-... --protocol tcp --port 443 --source-group sg-client`.
- **Scenario Context:** A blocked response path with a restrictive NACL causes intermittent connections even though the SG allows port 443.
- **Best Practices:** Keep NACLs simple, restrict administrative ports, and change controls through reviewable IaC.

### Q6. What is Auto Scaling and how does it use health checks?
- **One-liner:** Auto Scaling maintains desired capacity by launching or terminating instances based on health and policy.
- **Analogy:** It is a staffing manager adding workers during a rush and replacing absent workers.
- **Detailed Explanation:** An ASG uses a launch template, subnets, health checks, min/max/desired capacity, and target tracking or step policies. `aws autoscaling describe-auto-scaling-groups` shows state; ALB health checks can replace instances that are alive but unhealthy.
- **Scenario Context:** CPU rises with traffic, target tracking increases instances, then scales in after a cooldown.
- **Best Practices:** Use immutable images, spread across AZs, set scale-in protection only deliberately, and test termination behavior.

## SECTION B: MID-LEVEL

### Q7. How does an ALB differ from an NLB?
- **One-liner:** ALB is Layer 7 HTTP routing; NLB is Layer 4 TCP/UDP/TLS with high throughput and static IP options.
- **Analogy:** ALB reads envelopes and routes by address; NLB forwards sealed packages by port.
- **Detailed Explanation:** ALB supports host/path rules, redirects, WAF integration, and HTTP/2; NLB preserves source IP and handles non-HTTP protocols. Choose target groups and health checks per service.
- **Scenario Context:** `/api` and `/web` route to separate services behind one ALB, while a database proxy uses NLB TCP forwarding.
- **Best Practices:** Terminate TLS intentionally, set meaningful health endpoints, and monitor 4xx/5xx, latency, and target health.

### Q8. Explain CloudWatch metrics, logs, and alarms.
- **One-liner:** CloudWatch collects time-series metrics, log streams, and alarms that can notify or act.
- **Analogy:** Metrics are gauges, logs are a security camera archive, and alarms are the control room.
- **Detailed Explanation:** Publish custom metrics with dimensions; use Logs Insights queries such as `fields @message | filter status >= 500`; alarms evaluate periods and datapoints-to-alarm. Composite alarms reduce alert storms.
- **Scenario Context:** An alarm combines ALB 5xx and latency, pages the on-call, while logs identify the failing endpoint.
- **Best Practices:** Alert on user impact and saturation, retain logs intentionally, use UTC, and attach runbooks to alarms.

### Q9. When would you choose RDS Multi-AZ, read replicas, or Aurora?
- **One-liner:** Multi-AZ provides synchronous standby failover, replicas scale reads, and Aurora separates distributed storage from compute.
- **Analogy:** Multi-AZ is a backup operator ready to take over; a read replica is an extra cashier; Aurora is a shared resilient vault with replaceable counters.
- **Detailed Explanation:** Multi-AZ is for availability, not read scaling; replicas are asynchronous and may lag; Aurora offers cluster endpoints and reader endpoints. Check `ReplicaLag`, connections, and failover time.
- **Scenario Context:** A reporting workload moves to replicas while the writer remains protected by Multi-AZ.
- **Best Practices:** Test failover, automate backups/PITR, size connection pools, and verify application behavior after endpoint changes.

### Q10. What is the difference between SQS and SNS?
- **One-liner:** SQS is durable pull-based queuing; SNS is pub/sub fan-out to multiple subscribers.
- **Analogy:** SQS is a queue at a service counter; SNS is a radio broadcast heard by subscribed stations.
- **Detailed Explanation:** Producers send to SQS; consumers poll and delete after processing. Visibility timeout hides in-flight messages; DLQs capture repeated failures. SNS can publish to SQS, Lambda, HTTP, or email.
- **Scenario Context:** An order event publishes to SNS and fans out to billing, fulfillment, and analytics queues.
- **Best Practices:** Make consumers idempotent, size visibility timeout above processing time, monitor age/depth, and encrypt queues/topics.

### Q11. How do CloudFormation and Terraform manage AWS safely?
- **One-liner:** Both declare desired resources and reconcile changes; Terraform uses state while CloudFormation owns stacks natively.
- **Analogy:** They are building plans that let a construction crew reproduce a site and review changes before work.
- **Detailed Explanation:** Run `terraform plan` before `apply`; use remote encrypted state with locking. CloudFormation change sets preview stack updates. Avoid manually changing resources managed by either tool.
- **Scenario Context:** A reviewed plan adds an AZ without replacing production databases because lifecycle and dependencies are explicit.
- **Best Practices:** Pin providers/modules, isolate accounts and state, protect critical resources, and run drift detection regularly.

### Q12. What is AWS Organizations and a landing zone?
- **One-liner:** Organizations groups accounts and applies governance; a landing zone provides standardized secure account foundations.
- **Analogy:** It is a company headquarters with departments, shared policies, and a consistent office setup.
- **Detailed Explanation:** Use OUs, SCPs, consolidated billing, delegated administration, and central CloudTrail/Config. SCPs set maximum permissions; they do not grant access. Separate production, security, and logging accounts.
- **Scenario Context:** An SCP prevents disabling CloudTrail in workload accounts, while a security account receives findings.
- **Best Practices:** Keep break-glass access controlled, codify account vending, and test SCPs in a sandbox before broad rollout.

## SECTION C: SENIOR LEVEL

### Q13. Design a highly available multi-account AWS platform.
- **One-liner:** Isolate environments and duties by account, deploy across AZs, and centralize guardrails, identity, logs, and networking.
- **Analogy:** Separate buildings limit blast radius while a central security desk enforces policy and collects cameras.
- **Detailed Explanation:** Use Organizations/OUs, SSO roles, SCPs, Transit Gateway or controlled VPC peering, centralized CloudTrail/Config, private endpoints, and regional failover where justified. Define RTO/RPO before selecting active-active or warm standby.
- **Scenario Context:** A compromised workload account cannot alter audit logs or production IAM because those live in restricted accounts.
- **Best Practices:** Document failure domains, rehearse account recovery, minimize shared services, and measure controls rather than assuming them.

### Q14. How would you control AWS cost without harming reliability?
- **One-liner:** Attribute spend, remove waste, right-size capacity, and trade commitment discounts against forecasted baseline usage.
- **Analogy:** Cost optimization is fleet management: know which trip each vehicle serves before selling cars.
- **Detailed Explanation:** Tag accounts/resources, use Cost Explorer and CUR, rightsizing recommendations, Savings Plans, spot for interruptible workers, S3 lifecycle, and scheduled non-production shutdowns. Tie cost to unit metrics such as cost/request.
- **Scenario Context:** A sudden NAT Gateway bill reveals cross-AZ traffic and missing VPC endpoints; routing changes reduce both cost and latency.
- **Best Practices:** Treat cost as an SLO, enforce budgets, review architecture before commitments, and never use spot for stateful quorum without a recovery design.

### Q15. Explain AWS disaster recovery strategy selection.
- **One-liner:** Backup/restore, pilot light, warm standby, and multi-site active-active trade recovery time, cost, and complexity.
- **Analogy:** They range from storing blueprints to maintaining a fully staffed duplicate hospital.
- **Detailed Explanation:** Define RTO/RPO, replicate data with native or application mechanisms, automate infrastructure recreation, and test restore plus DNS/identity dependencies. A backup is not a DR plan until restoration is measured.
- **Scenario Context:** A regional outage triggers a runbook that restores encrypted backups into a prebuilt network and validates checksums before traffic shift.
- **Best Practices:** Test at least quarterly, include secrets and external integrations, document owners, and stop calling untested backups “recovery.”

### Q16. How do you secure a large AWS estate?
- **One-liner:** Use layered preventive, detective, and responsive controls with centralized identity, logging, and continuous evidence.
- **Analogy:** Security is a building with locks, cameras, alarms, and an incident team—not one strong door.
- **Detailed Explanation:** Combine IAM Identity Center, SCPs, KMS, private networking, GuardDuty, Security Hub, Inspector, Macie where appropriate, CloudTrail data events, Config rules, and automated remediation. Apply threat modeling to exposed paths.
- **Scenario Context:** GuardDuty detects an unusual role use; the response workflow quarantines the principal, preserves evidence, rotates secrets, and restores from known-good artifacts.
- **Best Practices:** Prefer short-lived credentials, immutable deployments, tested incident playbooks, and evidence-based exceptions with expiry dates.

## SECTION D: PRODUCTION SCENARIOS

### Q17. Production API latency doubled after a release. What happened and how do you fix it?
- **One-liner:** Correlate deployment, ALB/application metrics, traces, and dependencies before changing capacity.
- **Analogy:** Find which section of a relay race slowed rather than adding runners everywhere.
- **Detailed Explanation:** **What happened:** p95 latency rose after release. **Why:** a new query caused RDS CPU and connection-pool waits. **Fix:** 1) freeze rollout and compare versions; 2) inspect traces, Performance Insights, and slow queries; 3) rollback via deployment controller; 4) add the index/fix query in staging; 5) canary and verify p95/error rate. **Prevention:** query plans, load tests, tracing, and latency SLO gates.
- **Scenario Context:** ALB latency with normal target count points downstream; database wait events confirm the hypothesis.
- **Best Practices:** Preserve evidence, use reversible changes, communicate impact, and make rollback a tested command.

### Q18. An S3 bucket was accidentally exposed. Respond.
- **One-liner:** Contain public access, preserve evidence, identify exposure, rotate affected secrets, and prevent recurrence.
- **Analogy:** Close the open warehouse door, check what left, then change the locks and install an alarm.
- **Detailed Explanation:** **What happened:** a bucket policy allowed `Principal: *`. **Why:** an unreviewed IaC change bypassed organization guardrails. **Fix:** 1) enable account/bucket Block Public Access; 2) remove policy and ACL exposure; 3) inspect CloudTrail/data events and access logs; 4) notify security/legal; 5) rotate credentials found in objects; 6) restore policy from reviewed code. **Prevention:** SCPs, Config rules, CI policy tests, and sensitive-data scanning.
- **Scenario Context:** Do not delete logs or overwrite objects before forensics; preserve versions and timestamps.
- **Best Practices:** Use deny-by-default, separate public content paths, and treat exposure as an incident even if no abuse is found.

### Q19. A deployment caused an AZ-wide capacity failure. What is the recovery plan?
- **One-liner:** Stop the rollout, preserve service with remaining AZs, then repair capacity and deployment assumptions.
- **Analogy:** Close a damaged bridge and route traffic over inspected bridges while rebuilding.
- **Detailed Explanation:** **What happened:** a launch template requested an unavailable instance type in one AZ. **Why:** no capacity fallback was tested. **Fix:** 1) halt ASG refresh; 2) verify target health and capacity by AZ; 3) shift/ rebalance using alternate types/subnets; 4) restore desired capacity; 5) validate error budget and data consistency; 6) resume canary. **Prevention:** diversified instance types, capacity reservations where justified, and multi-AZ game days.
- **Scenario Context:** Keep enough healthy capacity before terminating old instances; never rely on a single type for a critical fleet.
- **Best Practices:** Use launch-template versioning, automated rollback, and explicit AZ-level dashboards.

### Q20. Cross-account deployment suddenly fails with AccessDenied. Diagnose it.
- **One-liner:** Trace the full IAM decision chain: caller, trust policy, identity policy, SCP, permission boundary, and resource policy.
- **Analogy:** A visitor can have a valid badge but still fail the building’s floor, department, or emergency rules.
- **Detailed Explanation:** **What happened:** CI could assume a role yesterday but not today. **Why:** an SCP or trust-policy condition changed. **Fix:** 1) identify caller and target role; 2) inspect CloudTrail event/error; 3) run `aws sts get-caller-identity`; 4) check trust, identity, boundary, SCP, and KMS/resource policies; 5) restore the smallest missing permission; 6) rerun a read-only test then deploy. **Prevention:** policy-as-code tests, change review, and monitored break-glass access.
- **Scenario Context:** The same action may be denied by KMS even after `AssumeRole` succeeds.
- **Best Practices:** Avoid broad wildcard permissions, log role assumptions, and make deployment roles environment-specific.

## Coverage Summary
- **Foundations:** Regions, IAM, storage, VPC, network controls, scaling.
- **Production:** Load balancing, observability, databases, messaging, IaC, governance.
- **Senior/Scenarios:** multi-account architecture, cost, DR, security, and incident response.
