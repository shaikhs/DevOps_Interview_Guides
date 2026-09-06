# DevOps Interview Guide Series

## Senior DevOps, SRE, Platform Engineering & Cloud Interview Preparation

A professionally structured collection of DevOps and SRE interview guides covering Linux, Git, cloud platforms, Infrastructure as Code, containers, Kubernetes, CI/CD, observability, security, networking, automation, testing, artifact management, and modern AI-assisted operations.

This resource is designed for engineers preparing for:

- DevOps Engineer interviews
- Senior DevOps Engineer interviews
- Site Reliability Engineer (SRE) interviews
- Platform Engineer interviews
- Cloud Infrastructure Engineer interviews
- DevSecOps Engineer interviews
- Cloud Architect and Infrastructure Architect interviews
- Technical Lead and Engineering Manager discussions

---

## Guide Philosophy

The objective is not memorization. Each answer is structured to build genuine understanding that survives technical follow-up questions.

Every question uses five layers of depth:

1. **One-liner** — a concise answer for quick recall
2. **Real-world analogy** — an intuitive explanation that makes the concept easier to remember
3. **Detailed explanation** — technical depth, commands, configuration, and examples
4. **Scenario context** — how the concept appears in production
5. **Best practices** — practical guidance based on reliability, security, operability, and maintainability

---

## Difficulty Levels

Every guide is organized into four interview levels:

### Junior — Foundations

Covers the essential concepts expected from engineers entering their first DevOps, cloud, or infrastructure role.

Typical areas include:

- Command-line fundamentals
- Core terminology
- Basic configuration
- Common tools and workflows
- Simple troubleshooting

### Mid-Level — Production Engineering

Covers the practical knowledge expected from engineers with approximately 2–5 years of experience.

Typical areas include:

- Production deployments
- Troubleshooting methodology
- Automation
- Reliability patterns
- Security controls
- Scaling and operational trade-offs

### Senior — Architecture and Strategy

Covers architecture, organizational scale, governance, cost, risk, and long-term platform decisions.

Typical areas include:

- Multi-account and multi-region design
- Reliability and disaster recovery
- Fleet-scale operations
- Platform standardization
- Security architecture
- Engineering productivity
- Cost and capacity management
- Technology trade-offs

### Production Scenarios — Incident and Failure Analysis

Scenario questions simulate the types of failures senior engineers are expected to handle in real environments.

Each scenario addresses:

1. What happened
2. Why it happened
3. How to investigate
4. Step-by-step remediation
5. How to prevent recurrence

---

## Guide Structure

```text
DevOps_Interview_Guides/
├── README.md
├── Core_Guides/
│   ├── 01_Linux_Interview_Guide.md
│   ├── 02_Git_Interview_Guide.md
│   ├── 03_AWS_Interview_Guide.md
│   ├── 04_Terraform_Interview_Guide.md
│   ├── 05_Ansible_Interview_Guide.md
│   ├── 06_Docker_Interview_Guide.md
│   ├── 07_Kubernetes_Interview_Guide.md
│   ├── 08_CICD_Interview_Guide.md
│   ├── 09_Monitoring_Interview_Guide.md
│   ├── 10_DevSecOps_Interview_Guide.md
│   ├── 11_GCP_Interview_Guide.md
│   └── 12_Azure_Interview_Guide.md
└── Extended_Guides/
    ├── 13_Networking_Interview_Guide.md
    ├── 14_Container_Ecosystem_Interview_Guide.md
    ├── 15_Kubernetes_Ecosystem_Interview_Guide.md
    ├── 16_Testing_Quality_Gates_Interview_Guide.md
    ├── 17_Artifact_Management_Interview_Guide.md
    ├── 18_Scripting_Languages_Interview_Guide.md
    ├── 19_CICD_Extended_Interview_Guide.md
    ├── 20_AI_DevOps_Interview_Guide.md
    └── 21_Chef_Puppet_Interview_Guide.md
```

---

# Core Guides

## 1. Linux Interview Guide

**File:** `Core_Guides/01_Linux_Interview_Guide.md`

Topics include:

- Linux filesystem hierarchy
- File permissions, ownership, links, and access control
- Processes, threads, signals, and process states
- systemd targets and boot troubleshooting
- Disk, inode, and filesystem troubleshooting
- Memory management and the OOM killer
- Load average, CPU, I/O wait, and performance analysis
- Linux networking diagnostics
- Package management and dependency conflicts
- Cron and systemd timers
- Fleet hardening and CIS-aligned baselines
- Kernel tuning and high-throughput services
- Centralized logging and retention
- Production CPU incidents
- Inode exhaustion
- Failed boots after patching
- Suspected host compromise

## 2. Git Interview Guide

**File:** `Core_Guides/02_Git_Interview_Guide.md`

Topics include:

- Distributed versus centralized version control
- Working tree, staging area, and repository
- Merge, rebase, and conflict resolution
- `.gitignore` and tracked secrets
- Fetch, pull, reset, revert, and cherry-pick
- Interactive rebase
- Git bisect
- Git hooks
- Submodules and alternatives
- Branching models: Git Flow, GitHub Flow, and trunk-based development
- Branch protection and merge queues
- Large repository performance
- Git LFS and partial clones
- Secrets-leak detection and remediation
- Legacy VCS migrations
- Force-push recovery
- CI merge discrepancies
- Missing fixes in releases
- Removing large files from Git history

## 3. AWS Interview Guide

**File:** `Core_Guides/03_AWS_Interview_Guide.md`

Topics include:

- AWS accounts, organizations, and landing zones
- IAM, roles, policies, and least privilege
- VPC architecture, subnets, routing, and connectivity
- EC2, Auto Scaling, and load balancing
- S3 security, lifecycle, and replication
- RDS, DynamoDB, caching, and data durability
- EKS and AWS-native container platforms
- CloudWatch and operational visibility
- Multi-region architecture and disaster recovery
- AWS cost optimization and governance
- Security boundaries and incident response
- Production failures involving IAM, networking, scaling, storage, and cost

## 4. Terraform Interview Guide

**File:** `Core_Guides/04_Terraform_Interview_Guide.md`

Topics include:

- Declarative Infrastructure as Code
- Providers, resources, variables, outputs, and locals
- State, remote backends, locking, and state security
- Modules and reusable platform components
- Plan, apply, destroy, and lifecycle behavior
- Drift detection and reconciliation
- Workspaces and environment separation
- Import and state migration
- Testing and policy-as-code
- CI/CD integration and approval workflows
- State corruption and recovery scenarios
- Safe refactoring of production resources

## 5. Ansible Interview Guide

**File:** `Core_Guides/05_Ansible_Interview_Guide.md`

Topics include:

- Inventories, playbooks, roles, and tasks
- Idempotence and desired state
- Variables, precedence, facts, and templates
- Handlers and service orchestration
- Secrets with Ansible Vault
- Dynamic inventory and cloud automation
- Parallelism, delegation, and rolling deployments
- Testing and linting
- Fleet configuration drift
- Safe patching and production recovery

## 6. Docker Interview Guide

**File:** `Core_Guides/06_Docker_Interview_Guide.md`

Topics include:

- Images, containers, layers, and registries
- Dockerfiles and build optimization
- Multi-stage builds
- Networking and storage
- Resource limits and isolation
- Container security
- Image scanning and supply-chain controls
- Docker Compose
- Logging and troubleshooting
- Container startup failures
- Image bloat and registry incidents
- Production container lifecycle management

## 7. Kubernetes Interview Guide

**File:** `Core_Guides/07_Kubernetes_Interview_Guide.md`

Topics include:

- Kubernetes architecture and control plane
- Pods, deployments, ReplicaSets, and StatefulSets
- Services, ingress, and networking
- ConfigMaps and Secrets
- Requests, limits, QoS, and scheduling
- Probes and application health
- RBAC and cluster security
- Storage and persistent volumes
- Autoscaling and capacity management
- Cluster upgrades and multi-tenancy
- CrashLoopBackOff debugging
- Pending pods and failed rollouts
- Node pressure and control-plane incidents

## 8. CI/CD Interview Guide

**File:** `Core_Guides/08_CICD_Interview_Guide.md`

Topics include:

- CI/CD principles and pipeline design
- Jenkins, GitHub Actions, and GitLab CI
- Build, test, package, deploy, and verification stages
- Pipeline artifacts and caching
- Environments and promotion strategies
- Blue/green and canary deployments
- Quality gates and approvals
- Secrets and identity in pipelines
- Pipeline reliability and runner management
- Rollback and release governance
- Broken pipelines after upgrades
- Failed deployments and unsafe promotion scenarios

## 9. Monitoring Interview Guide

**File:** `Core_Guides/09_Monitoring_Interview_Guide.md`

Topics include:

- Metrics, logs, traces, and events
- Prometheus architecture and PromQL
- Grafana dashboards and alerting
- Alert quality and noise reduction
- SLI, SLO, SLA, and error budgets
- Elasticsearch, Logstash, and Kibana
- High-cardinality metrics
- Capacity planning and saturation
- Observability for Kubernetes and cloud platforms
- Monitoring failures and blind spots
- MTTR reduction and incident investigation

## 10. DevSecOps Interview Guide

**File:** `Core_Guides/10_DevSecOps_Interview_Guide.md`

Topics include:

- Shift-left security
- Threat modeling
- SAST, DAST, SCA, and container scanning
- Secrets detection and management
- IAM and workload identity
- Software supply-chain security
- SBOMs and artifact signing
- Policy-as-code
- Vulnerability prioritization
- Security gates without excessive delivery friction
- Incident response and compromised pipeline scenarios

## 11. GCP Interview Guide

**File:** `Core_Guides/11_GCP_Interview_Guide.md`

Topics include:

- Google Cloud projects, folders, and organizations
- IAM and service accounts
- VPC networks and shared VPC
- Compute Engine and managed instance groups
- Cloud Load Balancing
- GKE architecture and operations
- Cloud Storage and Cloud SQL
- Pub/Sub and event-driven systems
- Cloud Monitoring and Logging
- Terraform on GCP
- Cost management and quota troubleshooting
- Regional failure and identity incidents

## 12. Azure Interview Guide

**File:** `Core_Guides/12_Azure_Interview_Guide.md`

Topics include:

- Azure subscriptions, management groups, and resource groups
- Microsoft Entra ID and managed identities
- VNets, subnets, NSGs, and private endpoints
- Virtual Machine Scale Sets
- Azure Kubernetes Service
- Azure Storage and Azure SQL
- Azure Monitor and Log Analytics
- Azure DevOps and deployment patterns
- Policy, governance, and cost management
- Availability zones and disaster recovery
- Production identity, networking, and scaling incidents

---

# Extended Guides

## 13. Networking Interview Guide

**File:** `Extended_Guides/13_Networking_Interview_Guide.md`

Topics include OSI/TCP-IP models, DNS, HTTP/HTTPS, TLS, routing, NAT, firewalls, load balancing, proxies, VPNs, CIDR, packet capture, latency, MTU, retransmissions, and production connectivity failures.

## 14. Container Ecosystem Interview Guide

**File:** `Extended_Guides/14_Container_Ecosystem_Interview_Guide.md`

Topics include Podman, Buildah, rootless containers, OCI standards, image builds, registries, container security, daemonless workflows, Podman Compose, Buildah pipelines, and migration from Docker.

## 15. Kubernetes Ecosystem Interview Guide

**File:** `Extended_Guides/15_Kubernetes_Ecosystem_Interview_Guide.md`

Topics include Helm, Istio, Argo CD, service meshes, GitOps, chart design, progressive delivery, traffic management, mTLS, observability, multi-cluster operations, and GitOps recovery scenarios.

## 16. Testing & Quality Gates Interview Guide

**File:** `Extended_Guides/16_Testing_Quality_Gates_Interview_Guide.md`

Topics include test strategy, unit/integration/contract tests, Playwright, end-to-end testing, SonarQube, code coverage, quality gates, flaky tests, test environments, security quality checks, and release confidence.

## 17. Artifact Management Interview Guide

**File:** `Extended_Guides/17_Artifact_Management_Interview_Guide.md`

Topics include Nexus, Artifactory, Harbor, package repositories, Docker registries, artifact immutability, retention, promotion, replication, signing, vulnerability scanning, cleanup, and registry outage recovery.

## 18. Scripting Languages Interview Guide

**File:** `Extended_Guides/18_Scripting_Languages_Interview_Guide.md`

Topics include Python, Groovy, YAML, HCL, shell automation, API clients, error handling, idempotence, parsing, CI/CD scripting, configuration safety, validation, retries, and destructive automation prevention.

## 19. CI/CD Extended Interview Guide

**File:** `Extended_Guides/19_CICD_Extended_Interview_Guide.md`

Topics include Tekton, CircleCI, CloudFormation, pipeline portability, reusable workflows, event-driven pipelines, infrastructure deployments, release orchestration, disaster recovery for CI, runner security, schema migrations, and progressive delivery.

## 20. AI + DevOps Interview Guide

**File:** `Extended_Guides/20_AI_DevOps_Interview_Guide.md`

Topics include GitHub Copilot, Backstage, PagerDuty, AI-assisted operations, incident summarization, runbook retrieval, AI governance, prompt security, model evaluation, cost controls, human approval, hallucination prevention, and safe automation.

## 21. Chef & Puppet Interview Guide

**File:** `Extended_Guides/21_Chef_Puppet_Interview_Guide.md`

Topics include configuration management, cookbooks, recipes, manifests, modules, resources, catalogs, agents, idempotence, roles, environments, data bags, Hiera, policy, drift, reporting, secrets, and safe fleet convergence.

---

## Current Content Status

The repository currently contains:

- **21 completed guides**
- **12 Core Guides**
- **9 Extended Guides**
- **20 questions per guide**
- **420 structured interview questions total**
- **4 difficulty categories per guide**
- **5 answer layers per question**
- **Production scenario coverage in every guide**

The current implementation prioritizes depth and practical interview quality. It is structured so each guide can be expanded independently with additional question sets without changing the folder organization or answer model.

> Note: The original long-term target describes a 1,485-question, 793+ page library. The current repository baseline contains 420 deep questions across 21 guides and can be expanded toward that target in future content batches.

---

## Recommended Study Plans

### 30-Day Plan

1. Linux fundamentals and troubleshooting
2. Git workflows and recovery
3. AWS or Azure/GCP fundamentals
4. Terraform and Ansible
5. Docker and Kubernetes
6. CI/CD pipeline architecture
7. Monitoring, SRE, and incident response
8. DevSecOps and supply-chain security
9. Networking and testing
10. Review production scenarios and practice explaining trade-offs

### 60-Day Senior-Level Plan

- Days 1–10: Linux, Git, networking, and scripting
- Days 11–20: AWS, GCP, Azure, and cloud governance
- Days 21–30: Terraform, Ansible, Docker, and container security
- Days 31–40: Kubernetes, Helm, Istio, and GitOps
- Days 41–47: CI/CD, release engineering, and quality gates
- Days 48–53: Monitoring, SLOs, incident response, and capacity planning
- Days 54–57: DevSecOps, artifacts, SBOMs, and policy-as-code
- Days 58–60: Senior scenarios, architecture trade-offs, and mock interviews

---

## How to Answer in an Interview

Use this response pattern for technical questions:

1. **Define the concept clearly.**
2. **Explain why it matters operationally.**
3. **Give a concrete implementation or command example.**
4. **Describe a production scenario or failure mode.**
5. **Close with trade-offs and best practices.**

For senior-level questions, explicitly discuss:

- Reliability and availability
- Security and least privilege
- Cost and capacity
- Operational complexity
- Failure modes
- Rollback and recovery
- Observability
- Team ownership and governance
- Short-term versus long-term trade-offs

Avoid presenting tools as isolated technologies. Strong answers connect tools to outcomes such as reduced MTTR, safer deployments, improved developer velocity, lower cloud spend, stronger security, or higher service availability.

---

## Interviewer Follow-Up Preparation

Expect follow-up questions such as:

- What happens when this fails?
- How would you roll it back?
- How do you monitor it?
- How does this scale?
- What is the security risk?
- What is the cost impact?
- What trade-off did you make?
- How would you test this before production?
- How would you debug it at 2 a.m. during an outage?
- How would you standardize this across 500 services?

Use the scenario sections to practice answering these without relying on memorized definitions.

---

## Related Workspace Materials

This guide collection complements the senior DevOps/SRE preparation material in the parent workspace, including:

- `Interview_Preparation_Senior_DevOps_SRE.md`
- `SRE_Methodology.md`
- `Advanced_SRE_Platform_Engineering_Roadmap.md`

---

## Suggested Future Expansion

To move from the current 420-question baseline toward the full 1,485-question target, expand each guide using the same format:

- Add more Junior fundamentals where topic breadth is high.
- Add deeper Mid-Level implementation questions.
- Add architecture and governance questions for Senior candidates.
- Add additional production scenarios with measurable impact and recovery decisions.
- Add answer variants for AWS, Azure, GCP, self-managed, and hybrid environments.
- Add practical exercises and take-home troubleshooting tasks.
- Add mock interview question sets by role and experience level.

The existing numbering and directory layout are intentionally stable so future expansions can be added without reorganizing the collection.

---

## License and Usage

This is a personal interview-preparation resource. Adapt the examples, commands, architectures, and scenarios to your own environment, cloud provider, security policies, and organizational practices.

Always validate commands in a safe non-production environment before using them on live infrastructure.
