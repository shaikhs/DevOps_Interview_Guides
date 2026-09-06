# Linux Interview Guide — DevOps/SRE Edition

> Part of the **21-Guide DevOps Interview Series** (Core Guides: 12 | Extended Guides: 9)
> Every answer follows the 5-layer depth model:
> **1) One-liner** — quick recall · **2) Analogy** — intuitive mental model · **3) Detailed Explanation** — commands & examples · **4) Scenario Context** — where this shows up in production · **5) Best Practices** — how experienced engineers apply it

---

## How to Use This Guide

- **Junior** → foundational concepts for first DevOps roles
- **Mid-Level** → production patterns for engineers with 2-5 years
- **Senior** → architecture, scale, and strategy decisions
- **Scenarios** → real-world debugging that separates good from great

---

## SECTION A: JUNIOR LEVEL

### Q1. What is the Linux file system hierarchy, and what do `/etc`, `/var`, `/opt`, `/usr` contain?

- **One-liner:** Linux organizes everything under a single rooted tree (`/`) where each top-level directory has a defined purpose (config, variable data, optional software, user programs).
- **Analogy:** Think of `/` as an office building — `/etc` is the admin office (settings/policies), `/var` is the mailroom (changing content like logs and mail), `/opt` is a rented external office (third-party software), `/usr` is the main workspace (shared programs and libraries).
- **Detailed Explanation:**
  - `/etc` — system-wide configuration files (e.g. `/etc/nginx/nginx.conf`, `/etc/passwd`, `/etc/fstab`). Should be static and version-controlled.
  - `/var` — variable data that changes at runtime: logs (`/var/log`), spool files (`/var/spool`), cached data (`/var/cache`).
  - `/opt` — self-contained third-party applications, typically installed outside the package manager (e.g. `/opt/splunkforwarder`).
  - `/usr` — user-space programs and libraries shared system-wide (`/usr/bin`, `/usr/lib`, `/usr/share`).
  - `/home` — user home directories; `/root` — root's home; `/tmp` — temporary files, often cleared on reboot; `/proc` and `/sys` — virtual filesystems exposing kernel/process state.
  ```bash
  du -sh /var/log/*        # find what's eating disk in logs
  cat /etc/os-release      # identify distro/version
  ```
- **Scenario Context:** Disk-full alerts almost always point to `/var/log` (runaway logs) or `/tmp`. Knowing the hierarchy tells you *where* to look without guessing.
- **Best Practices:** Never store application state in `/tmp` (can be cleared by tmpwatch/systemd-tmpfiles); keep custom app installs under `/opt` to avoid clashing with the package manager; put logs under `/var/log/<app>` with log rotation configured.

---

### Q2. What's the difference between a hard link and a symbolic link?

- **One-liner:** A hard link is another name for the same inode (same data, same disk location); a symlink is a separate file that just points to a path.
- **Analogy:** A hard link is like two people having the same house key — either can open the house, and the house exists as long as one key exists. A symlink is a sticky note with an address written on it — if the house is demolished, the note points to nothing.
- **Detailed Explanation:**
  ```bash
  ln original.txt hardlink.txt        # hard link: same inode number
  ln -s original.txt symlink.txt      # symlink: new inode, stores a path
  ls -li original.txt hardlink.txt    # same inode number confirms hard link
  ```
  - Hard links: cannot cross filesystems/partitions, cannot link directories (mostly), file persists until *all* hard links are removed.
  - Symlinks: can cross filesystems, can point to directories, become "dangling" if target is deleted.
- **Scenario Context:** Deployment tooling (e.g. Capistrano-style releases, or `current -> releases/v123`) uses symlinks to atomically switch which release is "live" — a single `ln -sfn` swap enables instant rollback.
- **Best Practices:** Use symlinks for release/version switching and config indirection; avoid hard links across environments where filesystem boundaries (e.g. separate EBS volumes) make them fail silently.

---

### Q3. Explain Linux file permissions (`rwx`) and how `chmod`/`chown` work.

- **One-liner:** Every file has an owner, group, and "others" permission triplet controlling read/write/execute, expressed as symbolic (`rwx`) or octal (`755`) notation.
- **Analogy:** Like a document with three access lists — you, your team, and everyone else — each with their own read/edit/run rights.
- **Detailed Explanation:**
  ```bash
  chmod 755 script.sh     # owner: rwx, group: r-x, others: r-x
  chmod u+x script.sh     # add execute for owner only
  chown deploy:devops app.war   # change owner and group
  umask 022                # default permission mask for new files
  ```
  - Special bits: SUID (`4xxx`) runs a binary as its owner (e.g. `passwd`), SGID (`2xxx`) inherits group on new files/dirs, sticky bit (`1xxx`) restricts deletion in shared dirs like `/tmp`.
- **Scenario Context:** A deployment fails with "permission denied" because the app runs as `appuser` but the artifact was unzipped as `root`. Fixing ownership (`chown -R appuser:appuser`) resolves it.
- **Best Practices:** Follow least privilege — avoid `chmod 777`; use groups to share access instead of loosening "others"; audit SUID binaries regularly as they're a common privilege-escalation vector.

---

### Q4. What is the difference between a process and a thread in Linux?

- **One-liner:** A process is an independent execution unit with its own memory space; a thread is a lightweight execution unit that shares memory with other threads in the same process.
- **Analogy:** A process is a separate house (own address, own utilities); threads are roommates sharing that same house (shared kitchen/resources) but doing different chores simultaneously.
- **Detailed Explanation:**
  ```bash
  ps -eLf | grep nginx      # show threads (L) per process
  top -H                    # top with threads shown
  cat /proc/<pid>/status | grep Threads
  ```
  - Processes are isolated via virtual memory; inter-process communication needs pipes, sockets, shared memory.
  - Threads share heap/global memory but have separate stacks; cheaper to create, but a crash/bug in one thread can corrupt shared state.
- **Scenario Context:** A Java app shows high CPU — `top -H` reveals it's a single GC thread pegging a core, not the whole JVM process, narrowing the investigation immediately.
- **Best Practices:** Use `ps -eLf` / `top -H` before blaming "the process" — thread-level visibility often reveals the real bottleneck (e.g. one lock-contended thread).

---

### Q5. What are the Linux runlevels / systemd targets, and how do you check what's running at boot?

- **One-liner:** Runlevels (SysV) define system states like multi-user or reboot; systemd replaces them with "targets" (e.g. `multi-user.target`, `graphical.target`).
- **Analogy:** Like phone "modes" — airplane mode, do-not-disturb, normal — each enabling a different set of services.
- **Detailed Explanation:**
  ```bash
  systemctl get-default            # default boot target
  systemctl list-units --type=target
  systemctl set-default multi-user.target
  systemctl list-unit-files --state=enabled   # services enabled at boot
  ```
- **Scenario Context:** A server boots into rescue mode after a botched GRUB/network config change — recognizing `emergency.target` vs `multi-user.target` tells you immediately this is a boot-level issue, not an app issue.
- **Best Practices:** Always test target-affecting changes (network, disk mounts in `/etc/fstab`) on a non-critical host first; a bad `fstab` entry can hang boot entirely.

---

### Q6. How do you check disk usage and find what's consuming space?

- **One-liner:** Use `df` for filesystem-level usage and `du` for directory/file-level breakdown.
- **Analogy:** `df` tells you how full each storage tank is; `du` tells you which containers inside are taking up the volume.
- **Detailed Explanation:**
  ```bash
  df -hT                          # human-readable, with filesystem type
  du -sh /var/* | sort -rh | head -10   # top 10 space consumers
  find / -xdev -size +500M -exec ls -lh {} \;   # large files on this filesystem only
  lsof +L1                        # deleted-but-open files still holding space
  ```
- **Scenario Context:** `df` shows 100% used but `du` on `/` doesn't add up — classic sign of a deleted-but-still-open file handle (a log file deleted while a process still writes to it). `lsof +L1` finds it; restarting/reloading the process frees the space.
- **Best Practices:** Always cross-check `df` against `du`; when they disagree, suspect open file handles; set up disk-usage alerting at 80%, not 95%, to leave room to react.

---

## SECTION B: MID-LEVEL

### Q7. Explain how Linux memory management works — free, buffers/cache, and OOM killer.

- **One-liner:** Linux aggressively uses "free" RAM for disk cache to speed up I/O, and it's usually safe — the kernel reclaims it instantly under pressure, but the OOM killer steps in when it truly can't free enough memory.
- **Analogy:** Free RAM used as cache is like a librarian keeping recently returned books on a nearby cart instead of the shelf — the moment a bigger stack of books arrives, the cart is cleared instantly to make room.
- **Detailed Explanation:**
  ```bash
  free -h                      # note "available" column, not just "free"
  cat /proc/meminfo | grep -i cache
  dmesg -T | grep -i "out of memory"
  sudo journalctl -k | grep -i oom
  cat /proc/<pid>/oom_score    # higher = more likely to be killed
  ```
  - `available` (not `free`) is the real "how much can a new app get" figure — it accounts for reclaimable cache/buffers.
  - The OOM killer scores processes by `oom_score` (memory usage, priority, `oom_score_adj`) and kills the worst offender when memory + swap are truly exhausted.
- **Scenario Context:** A monitoring alert fires "only 200MB free RAM" on a healthy box with 32GB total — this is a false alarm because `free` was checked instead of `available`, and 28GB was reclaimable page cache.
- **Best Practices:** Alert on `available` memory, not raw `free`; set `oom_score_adj` to protect critical processes (e.g. a database) from being killed first; investigate OOM kills via `dmesg`/`journalctl -k`, not just app logs.

---

### Q8. How do you troubleshoot high load average when CPU usage looks normal?

- **One-liner:** Load average includes processes waiting on I/O, not just CPU-bound ones, so high load with low CPU usually points to disk or network I/O contention.
- **Analogy:** Load average is the length of the line at a counter — a long line doesn't mean the cashier (CPU) is slow; it might mean everyone is stuck filling out a form (waiting on I/O) before reaching the counter.
- **Detailed Explanation:**
  ```bash
  uptime                     # load averages: 1m, 5m, 15m
  mpstat -P ALL 1            # per-core CPU incl. %iowait
  vmstat 1 5                 # 'b' column = processes blocked on I/O
  iostat -xz 1               # per-device %util, await, svctm
  ps -eo state,pid,cmd | grep '^D'   # processes in uninterruptible sleep (I/O wait)
  ```
- **Scenario Context:** Load average of 40 on an 8-core box with CPU at 15% utilization — `iostat` shows one EBS volume at 98% `%util` with high `await`; the root cause is a saturated disk, not CPU, and the fix is either scaling storage IOPS or finding the process hammering it.
- **Best Practices:** Never read load average in isolation — always cross-reference `%iowait` and per-device `iostat`; on cloud instances, check provisioned IOPS/throughput limits before assuming an application bug.

---

### Q9. Walk through how you'd debug a process that won't die even after `kill -9`.

- **One-liner:** If `kill -9` fails, the process is usually stuck in an uninterruptible kernel state (`D` state, often NFS/disk I/O) where signals can't be delivered until the underlying I/O completes.
- **Analogy:** It's like trying to fire someone who's currently locked inside a vault mid-transaction — you can issue the termination, but it won't take effect until they physically finish and step out.
- **Detailed Explanation:**
  ```bash
  ps -eo pid,stat,cmd | grep <pid>     # STAT column shows 'D' = uninterruptible sleep
  cat /proc/<pid>/stack                # kernel stack (needs root, shows where it's stuck)
  ls -l /proc/<pid>/fd                 # what it has open (NFS mount? device?)
  ```
  - `kill -9` (SIGKILL) can't interrupt a process blocked in kernel space waiting on hardware/NFS.
  - Zombie processes (`Z` state) are a different issue — the process already exited but the parent hasn't reaped it via `wait()`; killing the parent (or having it call `wait`) clears zombies, since a zombie itself can't be killed (it's already dead).
- **Scenario Context:** An app hung after an NFS mount became unreachable; `ps` showed `D` state, `kill -9` did nothing, and the only real fix was restoring NFS connectivity (or in the worst case, rebooting since the kernel thread was unkillable).
- **Best Practices:** Distinguish `D` (stuck in kernel/I/O) from `Z` (zombie, needs parent to reap) from a normal hang (needs `strace`/`gdb`); avoid NFS/network mounts for anything latency-sensitive; set mount options like `soft,intr,timeo` to avoid indefinite hangs (with the tradeoff of possible data inconsistency vs `hard`).

---

### Q10. How does the Linux network stack let you diagnose "service unreachable" issues?

- **One-liner:** Systematically walk the stack — interface → routing → firewall → listening socket → application — rather than guessing.
- **Analogy:** Like tracing why a phone call doesn't connect: check your phone has signal (interface), the number is dialed correctly (routing), no call blocking is set (firewall), and the other person's phone is actually on and ringing (listening socket).
- **Detailed Explanation:**
  ```bash
  ip addr show                      # interface up? correct IP?
  ip route get 8.8.8.8               # routing path
  ss -tlnp | grep :443                # is anything actually listening?
  sudo iptables -L -n -v              # or nft list ruleset — firewall rules
  curl -v telnet://<host>:<port>       # or: nc -zv host port
  tcpdump -i eth0 host <ip> and port <port>  # confirm packets arriving
  ```
- **Scenario Context:** App team reports "can't connect to our service" — `ss -tlnp` shows nothing listening on the expected port because the app crashed on startup; the "network issue" was actually an application failure, caught in under a minute by checking the socket first.
- **Best Practices:** Always confirm the socket is actually listening (`ss -tlnp`) before blaming network/firewall; use `tcpdump` when packets need to be proven to arrive or not; keep a standard "L1→L7" checklist so triage doesn't skip steps under pressure.

---

### Q11. What is the difference between `apt`/`yum` package management and how do you handle dependency conflicts?

- **One-liner:** Both resolve and install software plus dependencies from repositories, but `apt` (Debian/Ubuntu, `.deb`) and `yum`/`dnf` (RHEL/CentOS/Amazon Linux, `.rpm`) use different package formats and dependency resolvers.
- **Analogy:** Both are like grocery delivery apps that also fetch the required side ingredients for a recipe — they just source from different stores (repos) with different catalogs (package formats).
- **Detailed Explanation:**
  ```bash
  apt list --installed | grep <pkg>       # Debian/Ubuntu
  apt-cache policy <pkg>                  # available versions & repo priority
  dnf list installed <pkg>                # RHEL8+/Amazon Linux 2022+
  yum deplist <pkg>                       # dependency tree (older RHEL)
  rpm -qa --last | head                   # recently installed rpm packages
  apt-get install -f                      # attempt to fix broken dependencies
  ```
  - Version pinning: `apt-mark hold <pkg>` / `yum versionlock add <pkg>` prevents unwanted upgrades.
- **Scenario Context:** A security patch needs to go out fleet-wide, but one legacy app pins an old OpenSSL version — resolving this means either isolating that app (container/chroot) or scheduling a compatibility upgrade rather than blocking the patch for everyone.
- **Best Practices:** Pin critical package versions explicitly in IaC/config management rather than relying on "whatever apt installs today"; test package upgrades in a staging environment identical to prod; maintain an internal mirror/repo for reproducible builds.

---

### Q12. How do cron jobs work, and what are common pitfalls?

- **One-liner:** Cron runs scheduled commands based on a time pattern (`min hour day month weekday`), but jobs frequently fail silently due to environment differences from an interactive shell.
- **Analogy:** Cron is an alarm clock that fires tasks — but the "room" it wakes up in (its environment: `PATH`, env vars, working directory) is nearly empty compared to your normal login session.
- **Detailed Explanation:**
  ```bash
  crontab -l                       # list current user's cron jobs
  */15 * * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1
  ```
  - Cron's `PATH` is minimal (often just `/usr/bin:/bin`) — scripts relying on tools in `/usr/local/bin` fail unless you set `PATH` explicitly in the crontab or script.
  - No interactive shell profile (`.bashrc`) is sourced, so aliases/functions/env vars from your shell won't exist.
  - Overlapping runs: a job that takes longer than its interval can pile up.
- **Scenario Context:** A backup script works perfectly when run manually but fails "command not found" via cron — the root cause is `PATH` not including the directory of a tool (e.g. `aws` CLI installed via pip in a user directory).
- **Best Practices:** Always set `PATH` and other required env vars explicitly at the top of scripts (not relying on cron's inherited env); redirect output to a log file (`>> log 2>&1`) so failures are visible; use `flock` to prevent overlapping runs; prefer `systemd timers` over cron for better logging/dependency management on modern systems.

---

## SECTION C: SENIOR LEVEL

### Q13. How would you design a Linux host hardening baseline for a fleet of 500+ production servers?

- **One-liner:** Codify hardening as immutable, version-controlled configuration (CIS benchmark-aligned) applied via automation, not manual per-host tweaks.
- **Analogy:** Like building cars on an assembly line with a fixed safety spec, instead of hand-adjusting each car's brakes individually after it's already on the road.
- **Detailed Explanation:**
  - Baseline via CIS Benchmark or DISA STIG as the reference; automate enforcement with Ansible/Chef and drift detection.
  - Key controls: disable unused kernel modules and services, enforce SSH hardening (no root login, key-only auth, `AllowUsers`), configure `auditd` for syscall auditing, enforce SELinux/AppArmor in enforcing mode, kernel sysctl hardening (`net.ipv4.conf.all.rp_filter`, disable IP forwarding where not needed), centralized log shipping so hosts aren't the source of truth for audit trails.
  - Patch management: staged rollout rings (canary → 10% → 50% → 100%) with automated rollback criteria.
  ```bash
  ansible-playbook -i inventory hardening.yml --check   # dry run, diff mode
  aide --check                      # file integrity monitoring
  auditctl -l                       # active audit rules
  ```
- **Scenario Context:** After a security audit flags inconsistent SSH configs across the fleet, the fix isn't a one-time script — it's converting the baseline into an idempotent Ansible role with a scheduled compliance run (e.g. nightly) that auto-remediates and reports drift.
- **Best Practices:** Treat hardening config as code (peer-reviewed, versioned); run drift detection continuously, not just at provisioning time; stage patch rollouts with automated health checks and rollback gates; integrate compliance scanning (e.g. OpenSCAP) into CI so non-compliant images never reach prod.

---

### Q14. How do you approach kernel tuning for a high-throughput network service (e.g. a proxy handling 100k+ connections)?

- **One-liner:** Tune file descriptor limits, TCP stack parameters, and connection tracking so the kernel doesn't become the bottleneck before the application does.
- **Analogy:** No matter how good the restaurant's chefs (app) are, if the front door only lets 100 people queue at once (kernel limits), the restaurant will look "slow" for reasons that have nothing to do with cooking speed.
- **Detailed Explanation:**
  ```bash
  ulimit -n                                    # per-process FD limit
  sysctl net.core.somaxconn=65535               # backlog queue for accept()
  sysctl net.ipv4.ip_local_port_range="1024 65535"
  sysctl net.ipv4.tcp_tw_reuse=1                # reuse TIME_WAIT sockets
  sysctl net.netfilter.nf_conntrack_max=1048576 # connection tracking table size
  sysctl net.core.rmem_max / wmem_max           # socket buffer sizing
  ```
  - Check `conntrack` table exhaustion (`dmesg | grep conntrack`) as a silent connection-drop cause behind NAT/firewalls.
  - NUMA/IRQ affinity tuning matters at very high packet rates (pinning NIC IRQs to specific cores, `ethtool -L`/`-G` for queue counts).
- **Scenario Context:** A load balancer starts silently dropping connections at ~65k concurrent connections despite CPU/memory headroom — the root cause is `nf_conntrack_max` being hit, invisible in application metrics and only visible in kernel logs.
- **Best Practices:** Load-test to the actual target concurrency before go-live, not just average traffic; monitor kernel-level counters (conntrack, socket buffer drops, retransmits) alongside app metrics; document every sysctl change with the reasoning, since these are easy to silently "fix" once and forget.

---

### Q15. How do you architect centralized logging and log retention across a large Linux fleet while balancing cost and compliance?

- **One-liner:** Ship logs off-host immediately (never rely on local disk as the durable store), tier storage by access frequency, and enforce retention policy as code.
- **Analogy:** Like a bank not keeping the only copy of transaction records in the teller's desk drawer — records are sent to a secure vault immediately, with older records moved to cheaper long-term archive.
- **Detailed Explanation:**
  - Local agents (Filebeat/Fluent Bit) tail logs and forward to a pipeline (Logstash/Kafka) into a search/analytics tier (Elasticsearch) — hot tier for recent/searchable data, warm/cold tier or S3/Glacier for compliance retention.
  - Enforce structured logging (JSON) at the application level to avoid regex-parsing brittleness downstream.
  - Retention policy encoded via ILM (Index Lifecycle Management) or S3 lifecycle rules, not manual cleanup scripts.
  ```bash
  journalctl --vacuum-time=7d           # local retention on hosts (safety net only)
  curl -s localhost:9200/_ilm/policy    # verify ILM policy applied
  ```
- **Scenario Context:** Compliance requires 1 year of audit logs, but Elasticsearch storage costs balloon keeping everything hot — the solution is a hot (7 days, fast search) → warm (30 days) → cold/S3 (1 year, rarely queried) tiering strategy driven by ILM policies, cutting cost by 60%+ while still meeting the retention requirement.
- **Best Practices:** Never treat host-local logs as the durable source of truth — a terminated instance should never mean lost logs; automate retention/tiering rather than manual purges; validate that "compliance-required" logs are actually restorable from cold storage periodically (untested backups/archives are not backups).

---

### Q16. When would you choose to build a custom kernel/module vs. tuning userspace, and how do you evaluate that tradeoff?

- **One-liner:** Only go below userspace when you've proven (with data) that the kernel itself — not configuration, not the app — is the bottleneck, because kernel-level changes carry outsized operational and security risk.
- **Analogy:** You don't rebuild a car's engine because the radio (application) is glitchy — you only touch the engine once you've proven, with real diagnostics, that the engine itself is the limiting factor, because that job is riskier and harder to reverse.
- **Detailed Explanation:**
  - Exhaust userspace/sysctl tuning first (as in Q14) — most "kernel problems" are actually default limits that just need reconfiguration.
  - Legitimate custom-kernel cases: specialized hardware drivers, real-time scheduling requirements (`PREEMPT_RT`), or eBPF-based observability/security tooling where mainline doesn't yet expose the needed capability.
  - Cost side: custom kernels mean you own patching/CVE tracking yourself, lose vendor support paths, and increase fleet heterogeneity (harder incident response, since "which kernel version is this?" becomes a real question).
- **Scenario Context:** A trading-adjacent workload needs microsecond-level scheduling latency — after profiling proves standard `CFS` scheduler jitter is the actual limiter (not app-level GC pauses or I/O), `PREEMPT_RT` becomes justified; but for 95% of "our app feels slow" tickets, the fix is nowhere near this deep.
- **Best Practices:** Require profiling data (not intuition) before considering kernel-level changes; if you do go custom, treat the kernel build itself as a versioned, CI-tested artifact with a documented CVE-tracking and rebuild process; prefer eBPF-based tooling over full custom kernels when the need is observability, not scheduling/driver behavior.

---

## SECTION D: SCENARIO-BASED (Production Debugging)

### Q17. Scenario: A production server suddenly shows 100% CPU usage and the app is unresponsive. Walk through your triage.

- **One-liner:** Isolate whether it's a runaway process, kernel-level issue, or downstream contention — in that order, fastest checks first.
- **Analogy:** Like a doctor doing triage — check vitals first (top-level metrics), then narrow to the specific organ (process/thread) before ordering deep tests (strace/perf).
- **Detailed Explanation — What happened → why → fix → prevention:**
  - **What happened:** `top` shows one PID consuming 100% of a core; app requests are timing out.
  - **Why it happened:** Determine root cause layer by layer:
    ```bash
    top -H -p <pid>              # which thread specifically
    strace -p <pid> -c           # syscall breakdown — spinning on what?
    jstack <pid>                 # if JVM — thread dump for deadlock/busy-loop
    perf top -p <pid>             # hot functions if native/compiled code
    ```
    Common root causes found this way: an infinite retry loop hitting a dead dependency, a regex catastrophic backtracking on malicious/malformed input, GC thrashing from a memory leak, or a runaway cron/batch job colliding with live traffic.
  - **Step-by-step fix:** Confirm impact scope (one host vs. fleet) → if isolated, drain traffic from that host at the LB level → capture diagnostics (`jstack`/`perf`/core dump) *before* restarting, since restarting destroys the evidence → restart the process/host → monitor recovery.
  - **How to prevent it forever:** Add circuit breakers/timeouts around external calls (stops retry storms), add regression tests for the specific regex/input pattern if that was the cause, add per-thread CPU/latency SLOs to catch this before it's customer-visible, and capture automatic diagnostic snapshots (thread dumps) on threshold breach so the next incident doesn't require live debugging under pressure.
- **Best Practices:** Always capture a diagnostic snapshot before restarting — "turn it off and on again" destroys the only evidence of root cause; automate the triage commands above into a single incident-response script so this doesn't depend on someone remembering syntax at 3am.

---

### Q18. Scenario: A batch job that writes millions of small files is bringing a filesystem to a crawl, and `df` still shows plenty of free space. What's going on and how do you fix it?

- **One-liner:** You've likely exhausted inodes, not disk blocks — `df -i`, not `df -h`, is the metric that matters here.
- **Analogy:** It's like a parking lot (disk space) with plenty of open pavement, but you've run out of numbered parking spot signs (inodes) to assign to any more cars — the space exists, but there's no "slot ID" left to give it.
- **Detailed Explanation — What happened → why → fix → prevention:**
  - **What happened:** File writes start failing with "No space left on device" while `df -h` shows the filesystem is nowhere near full.
  - **Why it happened:** `df -i` reveals `IUse%` at 100% — the filesystem was formatted with a fixed inode table size (common on ext4), and a workload creating huge numbers of tiny files exhausted the inode count long before disk blocks filled up.
    ```bash
    df -i /data                       # check inode usage explicitly
    find /data -xdev -printf '.' | wc -c   # (careful on huge trees) count files
    ```
  - **Step-by-step fix:** Identify and clean up/archive the offending small files (e.g. tar them into fewer larger archives, freeing inodes); if the workload is expected to continue, move it to a filesystem better suited for many small files (XFS handles this better than ext4's fixed inode table) or reformat with a higher inode ratio (`mkfs.ext4 -i <bytes-per-inode>`).
  - **How to prevent it forever:** Add inode usage (`df -i`) to the same monitoring/alerting as disk space usage — most teams only alert on block usage and are blindsided by this; for workloads that inherently generate huge file counts, batch/archive them (e.g. write into tar/parquet files) rather than one-file-per-record; choose the filesystem type based on the actual I/O pattern (many small files vs. few large files) during initial provisioning.
- **Best Practices:** Never assume `df -h` free space means "no storage problem" — always pair it with `df -i`; design data pipelines to avoid unbounded small-file generation on disk (use object storage or batched formats for that pattern instead).

---

### Q19. Scenario: After a routine kernel/security patch and reboot, a critical server fails to come back up — it drops to an emergency/rescue shell. How do you recover it, and how do you prevent this in future patch cycles?

- **One-liner:** Boot into rescue/single-user mode, read the actual boot failure reason (usually `/etc/fstab` or an initramfs/driver mismatch), fix it, and re-test the fix before touching the rest of the fleet.
- **Analogy:** Like a car that won't start after an oil change — you don't guess and start swapping parts; you check exactly what error the dashboard is showing (often something as simple as a loose cap) before doing anything invasive.
- **Detailed Explanation — What happened → why → fix → prevention:**
  - **What happened:** Reboot after patching drops to `emergency.target` / a rescue prompt instead of completing boot.
  - **Why it happened:** Investigate systematically:
    ```bash
    journalctl -xb                       # boot log — find the actual failing unit
    systemctl --failed                   # which systemd units failed
    cat /etc/fstab                       # a stale/incorrect entry (e.g. removed volume) commonly stalls boot
    dmesg | less                          # driver/module load failures from the new kernel
    ```
    Typical causes: an `/etc/fstab` entry referencing a volume/mount that no longer auto-mounts correctly (missing `nofail` option), a new kernel version missing a required out-of-tree driver module, or a security patch tightening a config (e.g. SELinux policy) that breaks a required service.
  - **Step-by-step fix:** From rescue shell, mount the root filesystem read-write, correct the specific `fstab`/config issue identified in `journalctl -xb`, exit rescue mode and reboot to confirm a clean boot; if it's a bad kernel, boot the previous kernel via GRUB menu as an immediate rollback while investigating.
  - **How to prevent it forever:** Never patch/reboot the entire fleet at once — use a canary ring (1 host → wait/verify → 10% → 50% → 100%) so this exact failure is caught on one machine, not the whole production fleet; add `nofail` (and ideally `x-systemd.device-timeout`) to non-critical `fstab` entries so a missing mount degrades gracefully instead of halting boot; keep the previous kernel available in GRUB as an automatic rollback path; add a post-patch automated health check (boot success + service status) as a required gate before the rollout proceeds to the next ring.
- **Best Practices:** Patch in rings with automated rollback criteria, never patch->reboot 100% of a fleet in one action; always keep N-1 kernel bootable; treat `/etc/fstab` changes with the same review rigor as application config, since a single bad line can halt boot entirely.

---

### Q20. Scenario: You're told "the server is compromised" — a process is making outbound connections to an unfamiliar IP. Walk through your incident response.

- **One-liner:** Contain first (isolate network access) without destroying evidence, then investigate methodically — don't kill the process or reboot immediately, since that erases forensic data.
- **Analogy:** Like discovering a burglar in the house — you don't first clean up the crime scene; you secure the perimeter (contain), then carefully document what you find before touching anything, so you actually learn how they got in.
- **Detailed Explanation — What happened → why → fix → prevention:**
  - **What happened:** A monitoring/EDR alert or manual `netstat`/`ss` check reveals a process with an unexpected outbound connection to an unknown external IP.
  - **Why it happened / investigate first:**
    ```bash
    ss -tnp | grep <suspicious-ip>          # which process, which PID
    ls -l /proc/<pid>/exe                   # what binary is actually running (may be deleted from disk)
    cat /proc/<pid>/cmdline                  # full command line
    lsof -p <pid>                            # open files/sockets
    ps --forest -e                           # parent process chain — how was it launched?
    last -a; who                             # recent logins
    ```
    Prioritize containment: isolate the host from the network at the security-group/firewall level (not by shutting it down) so the attacker loses C2 access but forensic state (memory, running process) is preserved for investigation.
  - **Step-by-step fix:** Snapshot the disk/take a memory dump for forensics before any remediation; identify the entry vector (exposed service, leaked credential, vulnerable dependency) from logs and the process ancestry; rotate all credentials/secrets that were reachable from that host; rebuild the host from a known-clean image rather than trying to "clean" a compromised system in place; restore service from the clean rebuild.
  - **How to prevent it forever:** Patch the entry vector identified (e.g. exposed service, unpatched CVE, leaked key) fleet-wide, not just on the affected host; add egress filtering (deny-by-default outbound, allow-list only required destinations) so C2 callbacks are blocked even if a future compromise occurs; enable host-based intrusion detection (auditd rules, EDR) with alerting on anomalous outbound connections; run a full post-incident review to check for lateral movement to other hosts using the same credential/vector.
- **Best Practices:** Never "clean" a compromised host in place and put it back into service — always rebuild from a known-good image; isolate at the network layer rather than powering off, to preserve volatile forensic evidence; treat credential rotation as mandatory after any suspected compromise, not optional.

---

## Coverage Summary for This Guide

| Level | Questions | Focus |
|---|---|---|
| Junior | Q1–Q6 | Filesystem, permissions, processes/threads, boot targets, disk basics |
| Mid-Level | Q7–Q12 | Memory management, load average, process states, networking triage, package management, cron |
| Senior | Q13–Q16 | Fleet hardening, kernel tuning at scale, centralized logging architecture, kernel vs. userspace tradeoffs |
| Scenarios | Q17–Q20 | CPU spike triage, inode exhaustion, failed boot after patching, security compromise response |

**Next guide in the series:** `02_Git_Interview_Guide.md`

