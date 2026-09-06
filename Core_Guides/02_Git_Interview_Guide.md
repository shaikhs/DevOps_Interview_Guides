# Git Interview Guide — DevOps/SRE Edition

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

### Q1. What is Git and how is it different from a centralized VCS like SVN?

- **One-liner:** Git is a distributed version control system — every clone is a full repository with complete history, not just a working copy of a central server.
- **Analogy:** SVN is like a single library where you must check a book out from the front desk to make notes; Git is like everyone having their own complete photocopy of the entire library, free to annotate offline and merge notes back later.
- **Detailed Explanation:**
  ```bash
  git clone <url>       # full history downloaded locally, not just latest snapshot
  git log --oneline     # works fully offline
  ```
  - No single point of failure — any clone can become the new "source of truth" if the server is lost.
  - Enables cheap local branching/committing without network round-trips.
- **Scenario Context:** A team's SVN server goes down mid-sprint and nobody can commit; with Git, developers keep committing locally and push once the remote is back — zero productivity loss from the outage.
- **Best Practices:** Still designate one remote (e.g. GitHub/GitLab) as the canonical source of truth for CI/CD and collaboration, even though Git is technically decentralized — ambiguity about "which repo is real" causes chaos.

---

### Q2. Explain the three areas in Git: working directory, staging area (index), and repository.

- **One-liner:** Changes move through three states — working directory (edited files) → staging area (`git add`, what will be committed) → repository (`git commit`, permanent history).
- **Analogy:** Like packing a suitcase — clothes on your bed (working directory), clothes you've picked and placed in the suitcase (staging), and the suitcase actually zipped and checked in (committed).
- **Detailed Explanation:**
  ```bash
  git status                 # shows state across all three areas
  git add file.txt           # working dir -> staging
  git commit -m "message"    # staging -> repository (creates a snapshot)
  git diff                   # working dir vs staging
  git diff --staged          # staging vs last commit
  ```
- **Scenario Context:** A developer accidentally commits a half-finished, broken change because they ran `git add .` without reviewing `git diff --staged` first — understanding the staging area as a deliberate checkpoint prevents this.
- **Best Practices:** Always review `git diff --staged` before committing; use `git add -p` (patch mode) to stage changes in reviewable chunks rather than blindly staging entire files.

---

### Q3. What's the difference between `git merge` and `git rebase`?

- **One-liner:** Merge combines two branch histories with a new merge commit preserving both timelines; rebase replays your commits on top of another branch, producing a linear history.
- **Analogy:** Merge is like joining two rivers into one, where you can still see where each stream came from; rebase is like picking up your canoe and carrying it upstream to launch again from a new starting point, so it looks like you were always on that path.
- **Detailed Explanation:**
  ```bash
  git checkout feature
  git merge main              # creates a merge commit, keeps both histories intact
  # vs.
  git checkout feature
  git rebase main             # replays feature's commits on top of main, linear history
  ```
  - Merge is non-destructive and safe on shared branches.
  - Rebase rewrites commit hashes — never rebase a branch others have already pulled/built on, unless coordinated.
- **Scenario Context:** A team wants a clean, linear `git log` for easier bisecting and code review — they rebase feature branches onto `main` before opening a PR, but always merge (never rebase) into `main` itself once shared.
- **Best Practices:** Rule of thumb — rebase local/private branches to clean up history before sharing; merge (not rebase) once a branch is shared/public; never `git rebase` a branch that others have already based work on without explicit team coordination.

---

### Q4. What is a merge conflict and how do you resolve one?

- **One-liner:** A merge conflict occurs when Git can't automatically reconcile changes to the same lines/file between two branches, requiring manual resolution.
- **Analogy:** Like two editors independently rewriting the same paragraph of a shared document — the word processor can't guess which version is "right," so a human has to read both and decide.
- **Detailed Explanation:**
  ```bash
  git merge feature-branch
  # CONFLICT (content): Merge conflict in app.py
  git status                      # lists conflicted files
  # Edit file, resolve markers:
  # <<<<<<< HEAD
  # ...your version...
  # =======
  # ...their version...
  # >>>>>>> feature-branch
  git add app.py                  # mark as resolved
  git commit                      # complete the merge
  ```
  - `git mergetool` can launch a visual diff tool; `git merge --abort` backs out entirely if things go wrong.
- **Scenario Context:** Two developers both refactor the same function in parallel branches; merging surfaces a conflict, and resolving it requires understanding *both* intents, not just picking one side blindly (which can silently drop a bug fix).
- **Best Practices:** Never blindly accept "theirs" or "ours" without reading both sides; keep branches short-lived and rebase/pull frequently to minimize conflict size; communicate with the other author when a conflict touches logic (not just formatting).

---

### Q5. What does `.gitignore` do, and what happens if a file is already tracked?

- **One-liner:** `.gitignore` tells Git which untracked files/patterns to exclude from `git status`/`git add`, but it has no effect on files that are already tracked.
- **Analogy:** It's a "do not photograph" sign for a room — but if a photo was already taken and filed (tracked) before the sign went up, the sign doesn't retroactively remove that old photo.
- **Detailed Explanation:**
  ```bash
  # .gitignore
  *.log
  node_modules/
  .env
  ```
  ```bash
  git rm --cached secrets.env      # untrack a previously committed file
  echo "secrets.env" >> .gitignore # now prevent it from being re-added
  ```
- **Scenario Context:** A `.env` file with credentials was committed months ago before `.gitignore` was set up — adding it to `.gitignore` now does *not* remove it from history; it must be explicitly removed (and ideally the secret rotated, since it's permanently in git history until rewritten).
- **Best Practices:** Set up `.gitignore` at project creation, before the first commit, using language/framework-specific templates (github/gitignore); if a secret is committed, rotate the secret immediately — don't rely solely on history rewriting to "fix" it, since old clones/forks may retain it.

---

### Q6. What is the difference between `git fetch` and `git pull`?

- **One-liner:** `git fetch` downloads remote changes without touching your working branch; `git pull` fetches *and* immediately merges (or rebases) into your current branch.
- **Analogy:** `fetch` is checking your mailbox and bringing letters inside without opening them; `pull` is checking the mailbox *and* immediately opening and acting on every letter.
- **Detailed Explanation:**
  ```bash
  git fetch origin              # updates origin/main locally, safe, no changes to your files
  git log origin/main..main      # see how far behind/ahead you are
  git pull origin main           # fetch + merge (or rebase, with --rebase) into current branch
  git pull --rebase origin main  # avoid merge commits when pulling
  ```
- **Scenario Context:** Before starting risky work, an engineer runs `git fetch` first to inspect incoming changes with `git log`/`git diff` before deciding to merge, rather than blindly `git pull`-ing and being surprised by an unexpected auto-merge.
- **Best Practices:** Prefer `git fetch` + review + explicit merge/rebase in situations where you want control; configure `git pull --rebase` as default for cleaner history on personal feature branches; never `git pull` with uncommitted local changes you haven't checked (`git status` first).

---

## SECTION B: MID-LEVEL

### Q7. Explain Git branching strategies: Git Flow, GitHub Flow, and Trunk-Based Development. When would you choose each?

- **One-liner:** Git Flow uses long-lived develop/release/hotfix branches for scheduled releases; GitHub Flow uses short-lived feature branches merged directly to `main` with continuous deployment; Trunk-Based Development pushes small changes straight to `main` behind feature flags with minimal branching.
- **Analogy:** Git Flow is like a formal shipping schedule with staged customs checkpoints; GitHub Flow is like a fast courier that ships as soon as a package (PR) is ready; Trunk-Based is like a factory conveyor belt where everyone works directly on the same line, using switches (feature flags) to hide unfinished work.
- **Detailed Explanation:**
  - **Git Flow:** `main`, `develop`, `feature/*`, `release/*`, `hotfix/*`. Good for versioned software with scheduled releases (e.g. desktop apps, embedded firmware) where multiple versions must be supported in parallel.
  - **GitHub Flow:** `main` is always deployable; feature branches → PR → review → merge → deploy. Good for web apps/SaaS with continuous deployment.
  - **Trunk-Based Development:** Everyone commits small, frequent changes to `main` (or very short-lived branches merged same-day), using feature flags to hide incomplete features. Favored at high-scale orgs (Google, Meta-style) for minimizing merge conflicts and integration pain.
  ```bash
  git checkout -b feature/checkout-redesign main
  # ... work, small commits ...
  git push origin feature/checkout-redesign   # open PR into main
  ```
- **Scenario Context:** A SaaS company doing 10+ deploys/day outgrows Git Flow because the release-branch overhead adds friction with no real benefit — they migrate to GitHub Flow (or trunk-based with flags) to match their actual deployment cadence.
- **Best Practices:** Match the branching strategy to release cadence and product type, not habit — Git Flow for scheduled/parallel-version releases, GitHub Flow or trunk-based for continuous deployment; whichever is chosen, keep branches short-lived to minimize conflict risk.

---

### Q8. What is `git rebase -i` (interactive rebase) used for, and what are the risks?

- **One-liner:** Interactive rebase lets you edit, squash, reorder, or drop commits before they're shared — cleaning up messy history — but it rewrites commit hashes, so it's dangerous on shared branches.
- **Analogy:** It's editing a rough draft into a clean final manuscript before publishing — fine while it's still your private draft, but you can't rewrite a book that's already been printed and distributed.
- **Detailed Explanation:**
  ```bash
  git rebase -i HEAD~5
  # pick   -> keep commit as-is
  # squash -> combine with previous commit
  # reword -> change commit message
  # drop   -> remove commit entirely
  ```
  - Squashing multiple "WIP" commits into one meaningful commit before opening a PR improves reviewability and `git bisect` usefulness later.
  - `git push --force-with-lease` (safer than `--force`) is needed after rebasing an already-pushed branch, since history diverges from the remote.
- **Scenario Context:** A feature branch has 30 "fix typo" / "wip" commits before review — interactive rebase squashes them into 3 logical commits, making the PR reviewable and preserving a meaningful `git log` for future debugging.
- **Best Practices:** Only interactive-rebase commits that haven't been shared, or that you've explicitly coordinated rewriting with collaborators; use `--force-with-lease` instead of `--force` to avoid clobbering others' pushed work; squash noisy "wip" commits before requesting review, not after.

---

### Q9. How do `git cherry-pick`, `git revert`, and `git reset` differ, and when do you use each?

- **One-liner:** `cherry-pick` copies a specific commit onto another branch; `revert` creates a new commit that undoes a prior commit's changes (safe on shared history); `reset` moves the branch pointer, optionally discarding commits/changes (rewrites history — unsafe if already shared).
- **Analogy:** `cherry-pick` is photocopying one page from another book into yours; `revert` is adding a correction page that cancels out an earlier mistake without tearing pages out; `reset` is literally ripping pages out of the book, which is fine for your personal notebook but disastrous if others already have copies referencing those pages.
- **Detailed Explanation:**
  ```bash
  git cherry-pick <commit-sha>          # apply one commit onto current branch
  git revert <commit-sha>               # new commit that undoes it, history preserved
  git reset --soft HEAD~1               # undo commit, keep changes staged
  git reset --hard HEAD~1               # undo commit AND discard changes (destructive)
  ```
- **Scenario Context:** A critical hotfix was committed directly to a release branch but also needs to go into `main` — `git cherry-pick` brings just that one commit over without merging the entire release branch; separately, a bad commit already deployed to production is undone via `git revert` (not `reset`) so the fix itself is auditable in history rather than erased.
- **Best Practices:** Use `revert` (not `reset --hard`) on any branch that's shared, deployed, or has CI history depending on it — reverting is additive and auditable; reserve `reset --hard` for local, unpushed mistakes only; use `cherry-pick` sparingly for true one-off fixes, since overuse leads to divergent, hard-to-track branch histories.

---

### Q10. What is `git bisect` and how do you use it to find a regression?

- **One-liner:** `git bisect` performs an automated binary search through commit history to pinpoint exactly which commit introduced a bug.
- **Analogy:** Like finding which page in a 500-page book introduced a typo by checking the middle page, narrowing to the correct half each time, instead of reading page by page from the start.
- **Detailed Explanation:**
  ```bash
  git bisect start
  git bisect bad                     # current commit is broken
  git bisect good v1.2.0              # this earlier tag was known-good
  # Git checks out a midpoint commit; test it, then:
  git bisect good     # or
  git bisect bad
  # repeat until Git reports the exact first-bad commit
  git bisect reset                    # return to original branch/state
  ```
  - `git bisect run <script>` fully automates this if you have a script that exits non-zero on failure (e.g. a test suite).
- **Scenario Context:** A performance regression appears sometime in the last 200 commits with no clear cause — `git bisect run ./run_perf_test.sh` automatically narrows it to a single commit in ~8 steps (log2(200)) instead of manually testing dozens of commits.
- **Best Practices:** Keep a reliable, fast reproduction script/test to pair with `git bisect run` for full automation; maintain a history of tagged "known-good" releases so you always have a valid starting point for bisecting; keep commits atomic/small so bisect actually isolates a meaningful single change, not a giant mixed commit.

---

### Q11. How do Git hooks work, and what are common uses in a DevOps pipeline?

- **One-liner:** Git hooks are scripts that run automatically at specific points in the Git workflow (e.g. before commit, before push), used to enforce quality gates locally or on the server.
- **Analogy:** Like a security checkpoint at specific doors in a building — some checks happen before you leave your desk (pre-commit), others before you enter the main building (pre-receive on the server).
- **Detailed Explanation:**
  ```bash
  .git/hooks/pre-commit         # runs before a commit is created
  .git/hooks/pre-push           # runs before pushing to remote
  .git/hooks/commit-msg         # validate commit message format
  ```
  - Client-side hooks (pre-commit, commit-msg) aren't version-controlled by default and aren't enforceable on other machines unless distributed via a tool like **pre-commit** framework or **Husky** (JS) that installs them via a setup script.
  - Server-side hooks (`pre-receive`, `update`) on self-hosted Git servers can hard-block pushes (e.g. rejecting secrets, enforcing branch naming).
- **Scenario Context:** A team repeatedly leaks API keys in commits despite `.gitignore` — they add a `pre-commit` hook (via the `pre-commit` framework with a secrets-scanning plugin like `detect-secrets` or `gitleaks`) that blocks the commit locally, and a server-side `pre-receive` hook as a second line of defense in case the local hook is bypassed/uninstalled.
- **Best Practices:** Never rely solely on client-side hooks for security-critical enforcement (they can be skipped with `--no-verify` or simply not installed) — always pair with server-side enforcement (branch protection rules, pre-receive hooks, or CI checks that block merge); distribute hook setup via a version-controlled tool (pre-commit, Husky) rather than manual `.git/hooks/` copying.

---

### Q12. What are Git submodules, and what problems do they cause? What alternatives exist?

- **One-liner:** Submodules let you embed one Git repo inside another at a pinned commit, but they're notoriously finicky to keep in sync and are a frequent source of "works on my machine" issues.
- **Analogy:** A submodule is like including a photocopy of a specific chapter from another book, glued in at a specific edition — but if you forget to specify *which* edition, or the reader forgets to fetch that specific chapter separately, they end up with a blank page.
- **Detailed Explanation:**
  ```bash
  git submodule add https://github.com/org/lib.git libs/lib
  git submodule update --init --recursive     # commonly forgotten step after clone
  git clone --recurse-submodules <url>         # clone pulling submodules too
  ```
  - Pain points: forgetting `--init --recursive` leaves empty directories; submodules pin to a *commit*, not a branch, so updates require an explicit extra commit in the parent repo; nested submodules compound the complexity.
  - Alternatives: **package managers/artifact registries** (npm, Maven, PyPI-style versioned packages) for shared library code; **monorepo** with build-tool-level dependency graphs (Bazel, Nx, Turborepo) if teams want atomic cross-project changes; **git subtree** as a simpler (if less flexible) alternative that copies code without the pinning complexity of submodules.
- **Scenario Context:** New hires repeatedly get build failures because they clone without `--recurse-submodules` — the team eventually migrates the shared library out to a versioned internal npm package instead, eliminating the entire class of onboarding friction.
- **Best Practices:** Prefer a proper package/artifact registry over submodules for shared code whenever the language ecosystem supports it; if submodules are unavoidable, document the clone/update commands prominently and consider a `Makefile`/setup script that runs `submodule update --init --recursive` automatically.

---

## SECTION C: SENIOR LEVEL

### Q13. How would you design branch protection, code review, and merge policies for a 100+ engineer organization on a monorepo?

- **One-liner:** Codify protection as tiered policy (mandatory reviews, status checks, CODEOWNERS-based routing, and merge queues) rather than relying on individual discipline, since manual enforcement doesn't scale past a handful of engineers.
- **Analogy:** Like an airport with automated security lanes (CI checks) and designated gate agents for specific flights (CODEOWNERS) — you don't rely on every passenger to "just know" the rules; the system enforces them structurally.
- **Detailed Explanation:**
  - **Branch protection:** require PR + minimum approvals, require status checks (CI, security scan, lint) to pass, require up-to-date branch before merge, restrict force-pushes and direct pushes to `main`.
  - **CODEOWNERS:** route review requests automatically to the right team based on changed file paths, so ownership is explicit and reviewers aren't guessed at.
  - **Merge queues:** at scale, PRs can pass CI individually but break `main` when combined (semantic conflicts) — a merge queue (GitHub Merge Queue, or tools like Zuul/Bors) re-tests the *combined* state before actually landing each PR, serializing merges safely.
  - **Monorepo-specific:** path-based CI triggers (only run affected project's tests, not the whole repo) to keep feedback fast at scale; ownership boundaries enforced via CODEOWNERS + optionally custom pre-receive checks.
  ```yaml
  # CODEOWNERS example
  /services/payments/  @payments-team
  /infra/terraform/    @platform-team
  ```
- **Scenario Context:** As the org grows past ~50 engineers, "main is red" incidents spike because independently-passing PRs combine into breakage — introducing a merge queue eliminates this class of incident entirely by testing the actual merge result before landing, not just each PR in isolation.
- **Best Practices:** Automate policy enforcement structurally (branch protection + CODEOWNERS + merge queue) rather than relying on convention or trust; keep required status checks fast and reliable, since flaky/slow CI erodes trust in the gate and engineers start requesting bypasses; review and prune branch protection rules periodically as the org's risk profile changes.

---

### Q14. How do you handle Git repository performance at scale (huge repos, large binary files, thousands of contributors)?

- **One-liner:** Combat repo bloat and clone/fetch slowness with shallow clones, sparse checkouts, Git LFS for binaries, and potentially partial clone/monorepo tooling (e.g. Git's `partial clone` feature or moving to a purpose-built monorepo VCS).
- **Analogy:** Instead of shipping an entire warehouse's inventory to every new employee on day one, you ship only the shelf sections they actually work on, and store the heavy bulky items (binaries) in a separate depot they fetch on demand.
- **Detailed Explanation:**
  ```bash
  git clone --depth 1 <url>                      # shallow clone, only latest history
  git clone --filter=blob:none <url>              # partial clone: skip large blobs until needed
  git sparse-checkout set services/payments        # only materialize needed subdirectories
  git lfs track "*.psd"                            # Git LFS for large binaries
  ```
  - Git LFS replaces large binary blobs with pointer files in the actual Git history, storing the real content in separate LFS storage — keeps repo clone size manageable.
  - For truly massive monorepos (Google/Microsoft scale), teams often adopt virtual filesystem tooling (e.g. Microsoft's VFS for Git / Scalar) or split into multiple repos with a build-tool-level dependency graph (Bazel) instead of one giant Git repo.
  - Regular `git gc`/repacking on the server side keeps repository storage efficient.
- **Scenario Context:** CI clone times balloon to 10+ minutes on a monorepo with years of accumulated binary assets — introducing Git LFS for those assets plus `--filter=blob:none` partial clones in CI cuts clone time to under a minute, since CI no longer downloads full binary history it doesn't need.
- **Best Practices:** Never commit large binaries directly into Git history without LFS — once committed, they bloat every future clone forever unless history is rewritten; evaluate partial/shallow clone strategies specifically for CI (which rarely needs full history) vs. developer machines (which may need more); revisit "one repo vs many repos" architecture decisions as scale changes, since what works at 10 engineers often doesn't at 500.

---

### Q15. How would you design a secrets-leak prevention and remediation strategy across an org's Git usage?

- **One-liner:** Layer prevention (pre-commit scanning), detection (continuous repo/history scanning), and a documented incident response process (rotate + purge) — because any single layer alone will eventually fail.
- **Analogy:** Like airport security having a checkpoint (prevention), roaming security cameras (detection), and a documented emergency response plan (incident process) — relying on just the checkpoint means anything that slips through goes unnoticed indefinitely.
- **Detailed Explanation:**
  - **Prevention:** pre-commit hooks with `gitleaks`/`detect-secrets`/`trufflehog`, combined with server-side `pre-receive` hooks as a non-bypassable second layer; secrets management tooling (Vault, AWS Secrets Manager) so secrets never need to be hardcoded in the first place.
  - **Detection:** continuous scanning of the full repo history (not just new commits) via scheduled `gitleaks detect` runs or GitHub Advanced Security secret scanning, since a leak from 2 years ago in an old commit is just as exploitable as a new one.
  - **Incident response (documented runbook):** 1) rotate/revoke the exposed credential immediately (this matters far more than scrubbing history, since the secret must be assumed compromised the moment it's pushed, even to a private repo) 2) assess blast radius (what did that credential have access to, check access logs) 3) rewrite history to remove the secret (`git filter-repo` or BFG Repo-Cleaner) and force-push, coordinating with all clones/forks 4) notify affected teams.
  ```bash
  gitleaks detect --source . --report-path findings.json
  git filter-repo --path secrets.env --invert-paths     # remove file from all history
  ```
- **Scenario Context:** A scan of repo history (not just recent commits) surfaces an AWS key committed 18 months ago that's still technically valid — the response prioritizes immediate key rotation (minutes) over history cleanup (which can take longer to coordinate across forks), since rotation closes the actual exposure window.
- **Best Practices:** Treat credential rotation as the *first* and mandatory step, always faster and more important than history rewriting; scan full history periodically, not just new commits, since old leaks remain exploitable; make secrets management tooling (Vault/Secrets Manager) the default path so developers have no reason to hardcode credentials in the first place.

---

### Q16. How do you evaluate and migrate a legacy version control system (e.g. SVN, Perforce) to Git at an organizational scale?

- **One-liner:** Treat it as a phased migration with a proven conversion tool, a defined history-preservation policy, and a hard cutover date — running both systems indefinitely creates permanent confusion and drift.
- **Analogy:** Like relocating a company to a new office building — you don't leave half the departments in the old building indefinitely; you plan which records move (history), do a trial run with one team, then execute a firm moving day for everyone.
- **Detailed Explanation:**
  - Assess repo size/history depth and choose the right conversion tool: `git-svn` for SVN, `git-p4` for Perforce, or vendor-specific export tools; large repos may need history truncation decisions (full history vs. a defined cutoff with an archive of the old system kept read-only).
  - Pilot with one non-critical team first to catch tooling/workflow gaps (build scripts referencing SVN revision numbers, CI referencing old paths) before company-wide rollout.
  - Plan around the actual pain points of the *old* system that justified migrating (e.g. SVN's poor branching) by choosing an appropriate new branching strategy (Q7) — migrating tooling without changing workflow habits wastes the opportunity.
  - Set a hard cutover date with the legacy system made read-only immediately at cutover to prevent divergent history between the two systems.
- **Scenario Context:** An org migrating from Perforce to Git initially tries running both in parallel "during transition," and within two weeks changes diverge because engineers commit to whichever system is more convenient that day — the fix is a firm, communicated cutover: Perforce goes read-only the moment Git is live, with no ambiguity about which is canonical.
- **Best Practices:** Never run legacy and new VCS as dual sources of truth beyond a short, explicit pilot window; migrate build/CI tooling and internal documentation (wikis, runbooks referencing old commands) as part of the same project, not as an afterthought; use the migration as an opportunity to adopt better branching/review practices, not just a lift-and-shift of old habits into new tooling.

---

## SECTION D: SCENARIO-BASED (Production Debugging)

### Q17. Scenario: A developer accidentally force-pushed over `main`, wiping out several days of merged commits from the team. Walk through recovery.

- **One-liner:** Recover using `git reflog` (local) or the remote's own history/backup mechanisms — the "lost" commits usually still exist and aren't actually garbage-collected yet, they're just unreferenced.
- **Analogy:** It's like someone tearing pages out of the "table of contents" (the branch pointer) — the actual pages (commits) are still bound in the book; you just need to find and re-list them.
- **Detailed Explanation — What happened → why → fix → prevention:**
  - **What happened:** `git push --force origin main` overwrote `main` with an outdated/divergent local branch, discarding several days of teammates' merged commits from the remote's history.
  - **Why it happened:** Force-push was allowed on `main` (no branch protection blocking it), and the developer's local `main` was stale/diverged, likely from an old clone or a bad rebase.
  - **Step-by-step fix:**
    ```bash
    # On any teammate's machine that had fetched recently, or the remote's own logs:
    git reflog                                # find the SHA of main before the bad push
    git branch recovery-main <old-good-sha>
    git push origin recovery-main:main --force-with-lease   # restore, carefully
    ```
    - If no local machine has the pre-push state cached, check the Git hosting provider (GitHub/GitLab/Bitbucket) — most retain unreachable commits for a grace period and can be recovered via their support/API, or via `git fsck --unreachable` on a server-side mirror if you have SSH access to the bare repo.
    - Once restored, communicate to the team to re-fetch/reset their local `main` to match, since anyone who already pulled the bad force-push has a corrupted local `main` too.
  - **How to prevent it forever:** Enable branch protection on `main` disallowing force-pushes and direct pushes entirely (require PRs); this single setting change makes this entire incident structurally impossible going forward.
- **Best Practices:** Branch protection against force-push on shared branches should be a day-one repository setting, not a lesson learned after an incident; keep `reflog` in mind as the first recovery tool for *local* mistakes, and know your Git host's recovery window (typically 90 days for unreachable objects) as a safety net for *remote* mistakes.

---

### Q18. Scenario: CI suddenly starts failing on a PR with "merge conflict" even though the PR shows no conflicts in the GitHub UI. What's your investigation approach?

- **One-liner:** This usually indicates the PR's test/merge is being run against a stale base branch reference, or a submodule/LFS pointer mismatch that the UI's simple text-diff view doesn't surface.
- **Analogy:** It's like a restaurant reservation system showing a table as free while the kitchen's separate ticket queue still thinks it's occupied — two systems looking at overlapping state can disagree if one is stale.
- **Detailed Explanation — What happened → why → fix → prevention:**
  - **What happened:** CI's merge/build step reports a conflict; the GitHub PR page shows "no conflicts, ready to merge."
  - **Why it happened:** Investigate by reproducing locally exactly as CI does:
    ```bash
    git fetch origin
    git checkout -b test-merge origin/main
    git merge origin/feature-branch --no-commit --no-ff
    ```
    Common root causes: CI checks out a cached/stale `main` ref (didn't fetch latest before merging), a submodule pointer conflict (invisible in the top-level file diff the UI shows), Git LFS pointer file conflicts (binary-looking diff that the UI renders differently than raw text), or line-ending/`.gitattributes` normalization differences between the CI runner's OS and the developer's.
  - **Step-by-step fix:** Once the actual conflicting file/section is identified via local reproduction, resolve it directly (update the submodule pointer, re-run LFS track, or fix line-ending config), commit, and push; if it's a CI caching issue (stale base ref), clear the CI runner's git cache/workspace for that job.
  - **How to prevent it forever:** Ensure CI always does a fresh `fetch`/checkout of the actual target branch rather than relying on a cached workspace across runs; add `.gitattributes` with explicit line-ending rules (`* text=auto eol=lf`) so cross-OS contributors don't silently generate line-ending-only diffs; if submodules/LFS are in use, add a CI step that explicitly validates their state before the merge step.
- **Best Practices:** Always reproduce CI's exact merge steps locally before trusting a UI's simplified conflict view; treat "works differently than the UI shows" as a signal to check submodules, LFS, and line-ending normalization specifically, since those are the most common blind spots.

---

### Q19. Scenario: A production incident review reveals that a critical bug fix was merged to `main` weeks ago but somehow never made it into the last three production releases. How do you investigate, and how do you prevent this class of issue?

- **One-liner:** This is almost always a branching/release-process gap — the fix landed on `main` but the release branch/tag was cut from an older point, or a cherry-pick to the release branch was missed.
- **Analogy:** Like a corrected page being added to the master manuscript, but the printing press was still loading from an older, uncorrected proof — the master copy was right, but the thing actually shipped wasn't sourced from it.
- **Detailed Explanation — What happened → why → fix → prevention:**
  - **What happened:** `git log main` clearly shows the fix commit; production deploys from the last three releases don't include the behavior change.
  - **Why it happened:** Trace the actual lineage of what was deployed:
    ```bash
    git log --oneline main | grep <fix-commit-sha>       # confirm it's on main
    git branch --contains <fix-commit-sha>                # which branches actually include it
    git tag --contains <fix-commit-sha>                    # which release tags include it
    ```
    If using Git Flow-style release branches: the fix was merged to `main`/`develop` but the team was cherry-picking specific fixes into `release/*` branches manually, and this one was missed. If using GitHub Flow/trunk-based with tags: the release tag was cut from a commit predating the fix, possibly because the fix's PR was merged *after* the release branch/tag was created but the deploy pipeline used a cached/pinned ref instead of latest `main`.
  - **Step-by-step fix:** Identify the correct current release branch/tag, cherry-pick or merge the fix into it explicitly, redeploy, and verify via the actual running service (not just the git log) that the fix is live.
  - **How to prevent it forever:** If manual cherry-picking into release branches is part of the process, this is a structural risk — moving to a model where release branches/tags are always cut from `main` at deploy time (rather than manually maintained parallel branches) eliminates the "forgot to cherry-pick" failure mode entirely; alternatively, add an automated check that diffs "what's on main" vs "what's in the latest release tag" and flags divergence.
- **Best Practices:** Avoid manual, ad-hoc cherry-picking into long-lived release branches as a standing process — automate it or eliminate the need for it via trunk-based development with feature flags; add a post-release automated verification step comparing deployed commit SHA against expected `main`/tag state, so this kind of drift is caught within minutes of release, not weeks later during an incident review.

---

### Q20. Scenario: You need to permanently remove a large binary file (accidentally committed 500MB dataset) from a repository's entire history to fix repo bloat, without breaking the team's ongoing work. Walk through it.

- **One-liner:** Use `git filter-repo` (the modern, recommended tool, superseding `filter-branch`) to rewrite history removing the file everywhere, then coordinate a hard cutover since every commit hash after that point changes.
- **Analogy:** It's like discovering a giant unnecessary appendix bound into every single copy of a book ever printed — you can't just edit the master file going forward; you have to reprint every copy in circulation and get everyone to trade in their old copy for the new one.
- **Detailed Explanation — What happened → why → fix → prevention:**
  - **What happened:** A 500MB dataset file was committed months ago, and even though it was later deleted in a subsequent commit, it still exists in history — every clone still downloads it, causing multi-minute clone times and repo storage bloat.
  - **Why it happened:** Git tracks full history by design; deleting a file in a new commit doesn't remove it from earlier commits/blobs, so the object remains part of every future clone/fetch unless history itself is rewritten.
  - **Step-by-step fix:**
    ```bash
    pip install git-filter-repo   # or use BFG Repo-Cleaner as an alternative
    git filter-repo --path big_dataset.csv --invert-paths
    git push origin --force --all
    git push origin --force --tags
    ```
    - Coordinate the cutover: announce a freeze window, have all contributors push any pending work *before* the rewrite, then after the force-push, every contributor must re-clone (or carefully reset their local branches to match) since all commit hashes after the file's introduction have changed.
    - Run `git gc --prune=now --aggressive` on the server side afterward to actually reclaim the storage from now-unreferenced objects.
  - **How to prevent it forever:** Add the file type/pattern to `.gitignore` proactively for known large-data extensions; set up a pre-commit or pre-receive hook that rejects files above a size threshold (e.g. 10MB) before they ever enter history; for legitimate large files, use Git LFS from the start so they never bloat core history in the first place.
- **Best Practices:** History rewriting is a full-team, coordinated event, never a solo silent operation — communicate the freeze window and required re-clone/reset steps clearly; prefer preventing the problem (size-limit hooks, LFS) over needing this kind of surgical history rewrite after the fact, since rewriting is disruptive even when done correctly.

---

## Coverage Summary for This Guide

| Level | Questions | Focus |
|---|---|---|
| Junior | Q1–Q6 | Core concepts, three areas, merge vs rebase, conflicts, .gitignore, fetch vs pull |
| Mid-Level | Q7–Q12 | Branching strategies, interactive rebase, cherry-pick/revert/reset, bisect, hooks, submodules |
| Senior | Q13–Q16 | Branch protection at scale, repo performance/monorepo, secrets-leak strategy, legacy VCS migration |
| Scenarios | Q17–Q20 | Force-push recovery, phantom CI conflicts, missing fix in release, history rewrite for bloat |

**Previous guide:** `01_Linux_Interview_Guide.md`
**Next guide in the series:** `03_AWS_Interview_Guide.md`
