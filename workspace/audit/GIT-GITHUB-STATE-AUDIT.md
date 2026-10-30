# Git and GitHub state audit — workspace/audit

Executing agent (registered role of the agent that actually produced this work): DeepSeek
Report/handoff writer (if different from the executing agent): DeepSeek (same)
Executing-agent evidence (how the real executor was established): Work was executed in this session by the model identified in-session as `deepseek/deepseek-flash`; the control-plane `roles/DEEPSEEK.md` role was loaded and followed. The orchestrating CLI profile is not used as identity.
Report type / category (audit, reports, research, implementation, validation, handoffs): audit
Project ID / central report root: cross-project / control plane → `/home/iffi/dev-workspace/docs/workspace/`
Exact application repository / branch / SHA: N/A (workspace-wide audit)
Canonical report path `docs/<project>/<category>/<dated-name>.md` / SHA-256 / authorized remote evidence: `/home/iffi/dev-workspace/docs/workspace/audit/GIT-GITHUB-STATE-AUDIT.md`; SHA-256 recorded in "Report saved"; no remote evidence (local-only, read-only audit)

## Objective and exit criterion

Establish the actual Git/GitHub state of every repository and non-Git documentation area in
`/home/iffi/dev-workspace`, so the owner can decide how to version and sync everything to
GitHub. Read-only except for this report. No fixes, no commits, no pushes, no fetches, no
Qortium node contact, no Phase 2.

Exit criterion: every in-scope path inspected; per-repo branch/HEAD/upstream/ahead-behind/
remotes/dirty counts/stash/clean/branches recorded; GitHub connectivity and ownership
recorded; control-plane and docs secret-risk scan recorded by path+category only; repo
status identical before and after; only this report file created.

## Evidence by layer (command/action, environment, timestamp, result)

Audit timestamp: 2026-10-30T17:38:43Z (UTC), host `/home/iffi/dev-workspace`, git 2.47.3.
All commands were read-only: `git status`, `git log`/`rev-list`, `git branch`/`for-each-ref`,
`git remote -v`, `git stash list`, `git ls-remote`, `git config --get`, `du`, `find`, `grep -l`,
`ssh -T` probe, `gh` presence check. No fetch/pull/commit/push/stash/checkout/reset was run.

### Scope existence

| Path | Exists | Git repo? |
| --- | --- | --- |
| `/home/iffi/dev-workspace/control-plane` | yes | yes |
| `/home/iffi/dev-workspace/docs` | yes | no (no `.git` in path or any parent) |
| `/home/iffi/dev-workspace/projects/` (10 expected) | yes, all 10 present | each has `.git` |
| `/home/iffi/dev-workspace/scripts` | yes | no |
| `/home/iffi/dev-workspace/tools` | yes (234M: node, python-base) | no |
| `/home/iffi/dev-workspace/logs` | yes (empty) | no |
| `/home/iffi/dev-workspace/backups` | yes (14M) | not a repo at container level; contents intentionally not inspected |

`/home/iffi/dev-workspace` itself is **not** a Git repository and has no `.git` in any parent.

### Per-repository state (local refs only, no fetch)

Ahead/behind is `0 0` wherever an upstream exists. "Staged" = index entries; "Modified" =
unstaged tracked changes; "Untracked" = porcelain `??` files (recursive). Stash is empty for
every repository.

| Repository | Branch | HEAD | Upstream | Ahead/Behind | Remote (owner) | Mod | Untr | Stg | Clean |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `control-plane` | `phase1/control-plane` | `aef3e1ed45c66ef2382e5cc794665d1720f7ad77` | none | n/a | `github.com/iffinland/ai-development-orchestration` | 18 | 8 | 0 | no |
| `projects/nodefm-station` | `main` | `654d17f30065ed7a5c3ce0579ae386180128a5a0` | `origin/main` | 0 / 0 | `github.com/iffinland/nodefm-station` | 3 | 0 | 0 | no |
| `projects/Blogs` | `master` | `8ac557e751e84ebaf99ef13b1b97864af31e649c` | `origin/master` | 0 / 0 | `github.com/iffinland/Blogs` | 2 | 0 | 0 | no |
| `projects/Qortium-United-Community` | `main` | `703a5ac353c79910e8260b39db01207dc5d57122` | `origin/main` | 0 / 0 | `github.com/iffinland/Qortium-United-Community` | 2 | 0 | 0 | no |
| `projects/wordpress-to-qortal` | `main` | `f6eec4b655c3fb79c193cc6bd10562742dfe7bf1` | `origin/main` | 0 / 0 | `github.com/iffinland/wordpress-to-qortal` | 0 | 0 | 0 | yes |
| `projects/Qortal-Video-Bridge` | `main` | `3034c79d24dc7d2f7dfc38b3b014eb9371bb2926` | `origin/main` | 0 / 0 | `github.com/iffinland/Qortal-Video-Bridge` | 0 | 0 | 0 | yes |
| `projects/shadow-archives-webportal-QORTAL` | `main` | `7bbdc93fbef48db27228d6e5b34d7873aa773bc3` | `origin/main` | 0 / 0 | `github.com/iffinland/shadow-archives-webportal-QORTAL` | 1 | 0 | 0 | no |
| `projects/qortal-dev-workspace` | `main` | `b8fd2f6fbfcff88c0ab9bc8fd116d5548679d2d2` | `origin/main` | 0 / 0 | `github.com/iffinland/qortal-dev-workspace` | 0 | 0 | 0 | yes |
| `projects/ai-development-orchestration` | `agent/orchestration/bootstrap` | `c52e04841adfde062e579463a39bf0463609c8d5` | `origin/agent/orchestration/bootstrap` | 0 / 0 | `github.com/iffinland/ai-development-orchestration` | 0 | 0 | 0 | yes |
| `projects/qortium-home` | `main` | `a4b7ce113c33adb675aadd4e14c1b6d385570dfc` | `origin/main` | 0 / 0 | `github.com/QortiumDev/qortium-home` | 0 | 0 | 0 | yes |
| `projects/qortium-core` | `main` | `f0b688d80fb84753a32fcc897fa61e9106c9932c` | `origin/main` | 0 / 0 | `github.com/QortiumDev/qortium-core` | 0 | 0 | 0 | yes |

Local branches:
- `control-plane`: `phase1/control-plane` (HEAD, **not on remote**, no upstream),
  `agent/shadow-archives/qdn-grouped-publish-outcome-20261001` (tracks origin, same SHA `aef3e1e`).
- `projects/ai-development-orchestration`: `agent/orchestration/bootstrap` (HEAD, on remote).
- All other repositories: a single default branch (`main`, or `master` for `Blogs`) tracking origin.

Dirty working-tree detail (unstaged tracked changes only, no staged content anywhere):
- `control-plane` (18 M): `AGENTS.md`, `ENVIRONMENT.md`, `GIT-AND-HANDOFF.md`,
  `PROJECT-REGISTRY.md`, `README.md`, `WORKFLOW.md`, `roles/DEEPSEEK.md`, `skills/README.md`,
  eight `skills/qortal/*/SKILL.md`, `templates/REPORT.md`, `templates/TASK.md`.
  Untracked (4 top-level entries → 8 files): `adapters/`, `docs/`, `platforms/`,
  `tools/validate_control_plane.py`.
- `projects/nodefm-station` (3 M): `AGENTS.md`, `README.md`, `docs/ROADMAP.md`.
- `projects/Blogs` (2 M): `AGENTS.md`, `README.md`.
- `projects/Qortium-United-Community` (2 M): `AGENTS.md`, `README.md`.
- `projects/shadow-archives-webportal-QORTAL` (1 M): `AGENTS.md`.

Linked worktrees: none beyond the primary checkout in each repository (`git worktree list`
returned one entry per repo).

**Critical observation:** control-plane HEAD `aef3e1e` already exists on the remote as
`agent/shadow-archives/qdn-grouped-publish-outcome-20261001`, but the entire Phase 1/1.6
control-plane rewrite is **uncommitted** in the working tree. The local `phase1/control-plane`
branch is not published, and its remote (`iffinland/ai-development-orchestration`) is the same
repository used by `projects/ai-development-orchestration`. There is no dedicated
control-plane remote.

### GitHub connectivity and authentication (read-only)

- Global `user.name` / `user.email`: **unset**. No per-repository `user.*` override in any of
  the 11 repositories.
- Transport in use: **HTTPS** for every `origin` (`https://github.com/...`). No credential-bearing
  URL was found (no `user:pass@` and no userinfo `@` in any remote).
- `gh`: **not installed** (`gh: command not found`); `gh auth status` unavailable.
- Credential helper: none configured at system/global/local scope; no URL `insteadOf` rewrites.
  No `~/.git-credentials`; no `~/.config/gh`; no `GITHUB_TOKEN`, `GH_TOKEN`, `GIT_ASKPASS`, or
  `SSH_AUTH_SOCK`.
- SSH: not used by any remote. `~/.ssh` contains only `authorized_keys` (no client public/private
  keys). A non-mutating probe `ssh -T git@github.com` returned `Permission denied (publickey)`,
  confirming no usable SSH identity. The probe's transient `known_hosts` entry was removed to
  restore the pre-audit state.
- `git ls-remote --heads origin`: **success for all 11 repositories** (remote reachable and
  readable). Head counts: control-plane 8, nodefm-station 1, Blogs 1, Qortium-United-Community 1,
  wordpress-to-qortal 1, Qortal-Video-Bridge 1, shadow-archives-webportal-QORTAL 9,
  qortal-dev-workspace 6, ai-development-orchestration 8, qortium-home 17, qortium-core 3.
- Read access succeeded **without any configured credential**, which is consistent with public
  repositories or a transparently cached credential; visibility itself was not independently
  verified. **Write authentication is entirely unverified** — no token, helper, SSH key, or `gh`
  session is present, and no push was attempted.

### Remote ownership

| Owner | Repositories |
| --- | --- |
| `iffinland` | control-plane (via `ai-development-orchestration`), nodefm-station, Blogs, Qortium-United-Community, wordpress-to-qortal, Qortal-Video-Bridge, shadow-archives-webportal-QORTAL, qortal-dev-workspace, ai-development-orchestration |
| `QortiumDev` | qortium-home, qortium-core |

Write access to the `QortiumDev` repositories is **UNVERIFIED** (no evidence available; not
tested). Write access to `iffinland` repositories is likewise **unverified for this host**, since
no write credential exists in the current environment.

### Non-Git areas

- `docs/`: 216K, not a Git repository. Central documentation tree exists at
  `/home/iffi/dev-workspace/docs/`, containing project roots `Blogs`, `nodefm-station`,
  `Qortium-United-Community`, `shadow-archives-webportal-QORTAL`, `Qortal-Video-Bridge`,
  `wordpress-to-qortal`, plus `workspace`. No reports are stored at the `docs/` root.
- `scripts/`: 8.0K, only `env.sh`, not Git (env file contents not inspected).
- `tools/`: 234M, `node` (v24.22.2) plus `python-base`, not Git.
- `logs/`: empty, not Git.
- `backups/`: 14M, `agent-workspaces-20261030/` containing
  `opencode-global-config.pre-phase1.jsonc`, `qortal-dev-workspace/`,
  `ai-development-orchestration/`; contents intentionally **not inspected** per task rules.

### Secret-risk scan: `control-plane/` and `docs/` (paths + category only)

Method: pattern scans with `grep -l`/`grep -rl` so only file paths were emitted, never matched
values; credential-bearing-URL, private-key-block, token-format, assigned-secret key=value,
filename, and infrastructure-detail categories were checked. `.git` directories excluded.

| Category | Result |
| --- | --- |
| Private-key blocks (`BEGIN ... PRIVATE KEY`) | none |
| Known token formats (GitHub, Slack, AWS, generic `sk` prefixes) | none |
| Assigned secret-like `key=value` / `key: value` with a non-empty value | none |
| Credential-bearing URLs (URL userinfo containing a password) | none |
| `.env`, `*.pem`, `*.key`, `*.p12`, `*.pfx`, keystore/wallet, `id_rsa*`, `id_ed25519*`, `*credential*`, `*secret*` filenames | none |
| Mnemonic / seed phrase / keystore wording | `control-plane/platforms/README.md`, `docs/workspace/audit/AGENT-WORKSPACE-PHASE1.md` (prose guidance, no values) |
| Password/token/secret keyword occurrences | policy/prose and historical reports only; JSON hits are values/keywords, not assigned credentials |
| IPv4 / host-port infrastructure detail (review before publishing) | `control-plane/reports/2026-09-13-phase-2-reference-sync-and-tunnels.md`, `control-plane/reports/2026-09-13-phase-2-qortal-node-b-tunnel.md`, `control-plane/tasks/orchestration/qortal-skill-promotion-20260914/IMPLEMENTATION.md`, `control-plane/tasks/orchestration/qortal-skill-promotion-20260914/STATUS.json`, `control-plane/tasks/qwb-qortal-web-builders/qwb-rc-7fd03fc-staging-confirmation-20260916/TASK.md`, plus several `skills/qortal/*` and `skills/qortium/home2-qdn-publishing` files that cite API endpoints/examples |

Likely-secret assessment: **no hardcoded credentials, private keys, tokens, or wallet/keystore
files were detected** in `control-plane/` or `docs/`. The only sensitivity concern is **historical
infrastructure detail** (host/IP/tunnel context) in the older `control-plane/reports/` and
`control-plane/tasks/` artifacts, which the owner may wish to review or sanitize before
publishing to a public remote. Those are provenance archives under the report-storage policy.

`.gitignore` coverage: **no `.gitignore` exists** in `control-plane/` or `docs/` (nor in any
project root's central doc tree). `control-plane/.git/info/exclude` contains only the default
comments. No ignore rules currently protect `.env`/key/backup artifacts.

### `docs/` structure vs `REPORT-STORAGE-POLICY.md`

- Root layout matches the policy intent: one directory per registered project plus `workspace/`.
- No report exists directly in the `docs/` root; `workspace/README.md` and per-project
  `README.md` files describe routing.
- Category directories exist only where artifacts exist (`workspace/audit`, `workspace/reports`,
  `Blogs/validation`), which is consistent with the policy's "create on first artifact" rule.
- Minor deviations/notes: no `docs/README.md` at the root (not required by policy); existing
  audit filenames are mostly not date-prefixed (policy says "where appropriate"); the new
  `workspace/` README exists and points to the policy correctly.

## Files changed

- Created: `/home/iffi/dev-workspace/docs/workspace/audit/GIT-GITHUB-STATE-AUDIT.md` (this report).
- No other file in the workspace was created or modified. No repository working tree, index,
  branch, tag, stash, or remote-tracking ref was changed.

## Checks not executed and why

- No fetch/pull: prohibited and would mutate remote-tracking refs; ahead/behind uses local refs only.
- No push-based write-access test: prohibited; write access to `QortiumDev` (and `iffinland`) is
  reported UNVERIFIED.
- `gh auth status` not run: `gh` is not installed.
- No inspection of `backups/` contents or `scripts/env.sh`: task restricted backups and the scan
  scope to `control-plane/` and `docs/`.
- No Qortium node contact, no Phase 2.

## Adversarial self-audit

- Verified every in-scope path before inspecting; confirmed `docs/` and `dev-workspace` are not
  inside any Git repository.
- Compared per-repo `git status --porcelain` SHA-256 snapshots before and after; all 11 identical
  (see "Report saved" validation).
- Masking check: no remote URL contained credentials; none needed masking. No secret values were
  printed to the terminal or written here. The one file created by the SSH probe
  (`~/.ssh/known_hosts`) was deleted to restore the pre-audit state.
- Ambiguity recorded rather than guessed: `control-plane` has no upstream and its remote is a
  shared orchestration repository; write credentials are absent, so GitHub write state is
  explicitly unverified.
- Counts: untracked for control-plane reported as 8 files (4 porcelain directory entries);
  staged is 0 everywhere.

## External actions (commit/push/tag/release/deploy/QDN write/transaction)

None. No network write beyond read-only `git ls-remote` and the read-only SSH auth probe.

## Remaining risks and follow-up

- All substantive Phase 1/1.6 control-plane work is uncommitted and unpublished; a machine loss
  would lose it. (Not fixed here by instruction.)
- No Git write credentials exist on this host; GitHub sync cannot be performed until the owner
  provisions a credential (token or SSH key) and verifies write access.
- Two `QortiumDev` repositories have unverified write access.
- No `.gitignore` protects secrets/backups if `control-plane` or `docs` are later committed.
- Historical infrastructure details exist in old reports/tasks and may need review before public
  publication.

## Report saved

- Canonical absolute path: `/home/iffi/dev-workspace/docs/workspace/audit/GIT-GITHUB-STATE-AUDIT.md`
- Category: `audit`
- SHA-256: not embedded (a file cannot contain its own final hash); hash reported by the executor after write.
- Validation: per-repo `git status --porcelain` hashes identical before/after (all 11);
  only this report file created; no secret values present.

## Owner decisions needed

1. Which GitHub repository should own `control-plane`? Today it shares
   `iffinland/ai-development-orchestration` with `projects/ai-development-orchestration`, and the
   `phase1/control-plane` branch is unpublished. Should control-plane get a dedicated repo, stay
   on the orchestration repo, or live in a new branch there?
2. How should the existing uncommitted control-plane work (18 modified + 8 untracked files) be
   versioned — one checkpoint commit, a sequence of reviewable commits, or a dedicated branch?
   (No action taken.)
3. Where should the central `/home/iffi/dev-workspace/docs` tree live? A repo of its own, inside
   control-plane, or inside a project repo? Note `docs/` is currently outside all Git.
4. What must be excluded from commits: `backups/`, `logs/`, `tools/` (234M runtime), historical
   host/tunnel/IP details in `control-plane/reports` and `control-plane/tasks`, and any
   environment-specific files? Should `.gitignore` rules be added before the first commit?
5. Branch strategy: keep `phase1/control-plane` as a long-lived branch or merge to a default
   branch; and what branch naming/PR policy should apply to the app repos?
6. Which credential will be used for write access (fine-grained PAT, classic PAT, or SSH deploy
   key)? No write credential currently exists on this host; `QortiumDev` write access is
   unverified.
7. Are the `iffinland` repositories intended to be public? Anonymous HTTPS reads succeeded, so
   historical infrastructure detail would be publicly visible if committed there.
8. Should the remaining dirty app repos (`nodefm-station`, `Blogs`, `Qortium-United-Community`,
   `shadow-archives-webportal-QORTAL`) keep their local `AGENTS.md`/`README.md`/`ROADMAP.md`
   edits, and under whose review/commit authorization?
