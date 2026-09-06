# Ansible Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Five-layer depth model: **One-liner · Analogy · Detailed Explanation · Scenario Context · Best Practices**

## How to Use This Guide
- Q1–Q6 Junior; Q7–Q12 Mid-Level; Q13–Q16 Senior; Q17–Q20 production scenarios.

## SECTION A: JUNIOR LEVEL

### Q1. What is Ansible?
- **One-liner:** Ansible is agentless automation using YAML playbooks and remote modules.
- **Analogy:** A conductor gives instruments instructions without installing a conductor in every seat.
- **Detailed Explanation:** Control nodes connect mainly through SSH/WinRM; `ansible all -m ping -i inventory` tests reachability.
- **Scenario Context:** One playbook configures a fleet consistently.
- **Best Practices:** Use versioned playbooks and least-privilege accounts.
### Q2. What is an inventory?
- **One-liner:** Inventory defines managed hosts, groups, variables, and connection details.
- **Analogy:** A dispatch board grouping vehicles by route.
- **Detailed Explanation:** INI/YAML inventory groups hosts; dynamic plugins discover cloud instances; `ansible-inventory --graph` validates it.
- **Scenario Context:** Web servers receive a different role from databases.
- **Best Practices:** Separate environment inventories and avoid hard-coded secrets.
### Q3. What is a module?
- **One-liner:** A module performs one idempotent operation on a target.
- **Analogy:** A specialized tool rather than a generic shell hammer.
- **Detailed Explanation:** `ansible.builtin.package`, `service`, `copy`, and `command` communicate desired work and return facts.
- **Scenario Context:** `package` reports no change when a version is already present.
- **Best Practices:** Prefer modules over shell commands.
### Q4. What is a playbook?
- **One-liner:** A playbook maps hosts to ordered tasks, variables, handlers, and roles.
- **Analogy:** A repeatable run sheet for a technical production.
- **Detailed Explanation:** `ansible-playbook site.yml --check --diff` previews changes; plays target groups and tasks invoke modules.
- **Scenario Context:** A release play updates config then restarts only changed services.
- **Best Practices:** Lint YAML and keep plays small.
### Q5. What does idempotence mean?
- **One-liner:** Repeating automation converges to the same desired state without unnecessary changes.
- **Analogy:** Setting a thermostat to 21°C repeatedly does not keep changing the room.
- **Detailed Explanation:** `state: present` and `enabled: true` express outcomes; `shell` often needs `creates` or `changed_when`.
- **Scenario Context:** Re-running after a network timeout safely completes missing work.
- **Best Practices:** Test second-run output: ideally `changed=0`.
### Q6. What are variables and facts?
- **One-liner:** Variables provide inputs; facts are discovered target attributes.
- **Analogy:** Variables are instructions; facts are measurements taken before work.
- **Detailed Explanation:** `ansible_facts['os_family']` supports conditional tasks; precedence is complex, so use clear scopes.
- **Scenario Context:** Debian and Red Hat use different package names.
- **Best Practices:** Validate required variables and avoid deep precedence tricks.

## SECTION B: MID-LEVEL
### Q7. What are roles?
- **One-liner:** Roles package defaults, vars, tasks, handlers, templates, files, and metadata.
- **Analogy:** A reusable service kit with labeled compartments.
- **Detailed Explanation:** `roles/web/tasks/main.yml` is invoked with `roles: [web]`; dependencies compose roles.
- **Scenario Context:** The same hardened Nginx role serves multiple products.
- **Best Practices:** Keep interfaces documented and test with Molecule.
### Q8. How do handlers work?
- **One-liner:** Handlers run once at a controlled point when notified by changed tasks.
- **Analogy:** A mechanic is called only when a repair actually happened.
- **Detailed Explanation:** A template `notify: restart nginx`; handler runs at play end or `meta: flush_handlers`.
- **Scenario Context:** Ten config changes cause one restart instead of ten.
- **Best Practices:** Make handlers safe and schedule disruptive restarts deliberately.
### Q9. Explain `become`.
- **One-liner:** `become` escalates execution, commonly through sudo.
- **Analogy:** A worker borrows a supervisor’s key for one task.
- **Detailed Explanation:** `become: true` and `become_user: root` apply privilege to tasks; sudo policy controls actual permissions.
- **Scenario Context:** SSH uses an unprivileged deploy user while package installation escalates.
- **Best Practices:** Narrow sudo rules and never put passwords in playbooks.
### Q10. How do templates differ from copy?
- **One-liner:** `template` renders Jinja variables; `copy` transfers static content.
- **Analogy:** A printed form versus a blank template filled per customer.
- **Detailed Explanation:** `template: src=app.conf.j2 dest=/etc/app.conf`; use `validate` before replacement when supported.
- **Scenario Context:** Each host receives its own listener and backend values.
- **Best Practices:** Keep secrets out of rendered logs and use atomic writes.
### Q11. What is Ansible Vault?
- **One-liner:** Vault encrypts sensitive YAML values or files for repository storage.
- **Analogy:** A locked envelope inside the filing cabinet.
- **Detailed Explanation:** `ansible-vault encrypt group_vars/prod/secrets.yml`; CI supplies a protected password or secret manager integration.
- **Scenario Context:** A database credential is encrypted at rest but decrypted during the task.
- **Best Practices:** Rotate keys, restrict access, and remember output/state can still leak secrets.
### Q12. How do you control rollout risk?
- **One-liner:** Use serial batches, health checks, retries, and explicit failure behavior.
- **Analogy:** Renovate one apartment block while keeping others open.
- **Detailed Explanation:** `serial: 10%`, `max_fail_percentage`, `until`, `retries`, and `delegate_to` support controlled operations.
- **Scenario Context:** A bad package affects only the first canary batch.
- **Best Practices:** Drain traffic before changes and verify after each batch.

## SECTION C: SENIOR LEVEL
### Q13. Design Ansible at fleet scale.
- **One-liner:** Separate inventories and roles, use execution environments, and standardize CI and policy.
- **Analogy:** A logistics network with regional depots and a common toolchain.
- **Detailed Explanation:** Use Automation Controller/AWX, dynamic inventory, signed collections, RBAC, logs, and execution environments with pinned dependencies.
- **Scenario Context:** Teams run approved roles without installing conflicting Python libraries.
- **Best Practices:** Measure convergence, duration, failure rate, and ownership.
### Q14. How do you make Ansible secure?
- **One-liner:** Minimize privileges, protect variables, verify sources, and audit execution.
- **Analogy:** Give each technician only the keys for assigned rooms.
- **Detailed Explanation:** SSH keys, Vault/secret-manager lookup, `no_log: true` for secret tasks, signed content, and controller RBAC reduce exposure.
- **Scenario Context:** A compromised play cannot read every host’s credentials.
- **Best Practices:** Treat `no_log` as a trade-off and retain safe audit metadata.
### Q15. How do you test roles?
- **One-liner:** Lint, syntax-check, converge in disposable systems, verify, and destroy.
- **Analogy:** Test a repair kit in a lab before sending it to every branch.
- **Detailed Explanation:** `ansible-lint`, Molecule scenarios, Testinfra, and CI test multiple OS versions and second-run idempotence.
- **Scenario Context:** A role passes on Ubuntu but fails on RHEL due to service naming.
- **Best Practices:** Pin collections and test failure paths.
### Q16. When should Ansible not be used?
- **One-liner:** Avoid it for high-frequency application orchestration or immutable workloads better rebuilt than mutated.
- **Analogy:** Do not repeatedly remodel disposable pop-up stores.
- **Detailed Explanation:** Image baking, Kubernetes controllers, or Terraform may better manage immutable infrastructure; Ansible remains useful for bootstrap and configuration.
- **Scenario Context:** Rebuilding an image is safer than changing hundreds of live servers.
- **Best Practices:** Choose the control loop that matches resource lifecycle.

## SECTION D: PRODUCTION SCENARIOS
### Q17. A playbook changed every host unexpectedly.
- **One-liner:** Stop rollout, identify non-idempotent task, restore safely, and add a second-run gate.
- **Analogy:** A calibration tool reset every machine because it measured incorrectly.
- **Detailed Explanation:** **What happened:** a shell task always returned changed and rewrote config. **Why:** no idempotence guard. **Fix:** 1) halt job; 2) inspect diff; 3) restore known-good config; 4) replace shell with module/`creates`; 5) canary; 6) run twice. **Prevention:** lint and idempotence CI.
- **Scenario Context:** Preserve one affected host for debugging before mass repair.
- **Best Practices:** Require `--check --diff` for reviewable changes.
### Q18. A rolling upgrade caused an outage.
- **One-liner:** Restore capacity, stop the batch, and make health-aware serial deployment.
- **Analogy:** Too many buses left the station at once.
- **Detailed Explanation:** **What happened:** serial was too large and traffic was not drained. **Why:** availability budget was ignored. **Fix:** 1) pause; 2) rollback package/config; 3) restore healthy nodes; 4) drain and upgrade smaller batches; 5) verify endpoint; 6) resume. **Prevention:** capacity checks and canaries.
- **Scenario Context:** A playbook’s success status is not the same as application health.
- **Best Practices:** Integrate load-balancer drain and synthetic checks.
### Q19. Vault secrets appeared in job logs.
- **One-liner:** Contain exposure, rotate secrets, and fix logging controls.
- **Analogy:** A locked envelope was photographed while being opened.
- **Detailed Explanation:** **What happened:** a debug task printed a variable. **Why:** `no_log` was absent and verbose CI was enabled. **Fix:** 1) restrict logs; 2) revoke/rotate; 3) remove artifacts; 4) add `no_log`; 5) test redaction. **Prevention:** secret scanning and log retention controls.
- **Scenario Context:** Assume any CI viewer may have seen the value.
- **Best Practices:** Prefer short-lived credentials.
### Q20. Hosts are unreachable during a patch window.
- **One-liner:** Stop retries, distinguish network from host failure, and resume with bounded recovery.
- **Analogy:** Confirm whether the road or the destination is closed before sending more trucks.
- **Detailed Explanation:** **What happened:** SSH timeouts followed a firewall change. **Why:** the play changed network rules before validation. **Fix:** 1) stop play; 2) use console/out-of-band access; 3) restore rule; 4) test `ansible -m ping`; 5) continue with serial; 6) audit changes. **Prevention:** prechecks, rollback access, and one-host canary.
- **Scenario Context:** Keep an independent management path.
- **Best Practices:** Never patch the only access route first.

## Coverage Summary
- Ansible foundations, idempotence, roles, secrets, testing, scale, rollout safety, and incidents.
