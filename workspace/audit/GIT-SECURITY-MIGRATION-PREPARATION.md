# Git security and repository migration preparation — workspace/audit

Executing agent (registered role of the agent that actually produced this work): DeepSeek
Report/handoff writer (if different from the executing agent): DeepSeek (same)
Executing-agent evidence (how the real executor was established): Work was executed in this session by the model identified in-session as `deepseek/deepseek-flash`; `control-plane/roles/DEEPSEEK.md` and the control-plane contract were read and followed. The CLI/orchestration profile is not used as identity.
Report type / category (audit, reports, research, implementation, validation, handoffs): audit
Project ID / central report root: cross-project / control plane → `/home/iffi/dev-workspace/docs/workspace/`
Exact application repository / branch / SHA: control-plane `phase1/control-plane` @ `aef3e1ed45c66ef2382e5cc794665d1720f7ad77`; central docs: N/A (not a Git repository yet)
Canonical report path `docs/<project>/<category>/<dated-name>.md` / SHA-256 / authorized remote evidence: `/home/iffi/dev-workspace/docs/workspace/audit/GIT-SECURITY-MIGRATION-PREPARATION.md`; SHA-256 computed after write (not embedded); no remote evidence (local-only, read-only preparation)

## Objective and exit criterion

Prepare the existing control-plane and central documentation for migration into
the two new private GitHub repositories `iffinland/dev-control-plane` and
`iffinland/dev-workspace-docs`, without committing, pushing, changing remotes,
initializing the documentation repository, or altering GitHub state.

Exit criterion: governance read; current Git/worktree state recorded; central
docs structure and migration conflicts checked; public/private infrastructure
detail and credentials audited without printing values; safe local `.gitignore`
and security-validator files created; existing and new validators run with
pre-existing failures separated from task-introduced failures; a complete
migration sequence and exact future Git operations documented; no destructive
or architectural change performed.

Result: **PARTIAL**. Preparation is complete and safe, but one pre-existing
public infrastructure literal remains in a historical report and in existing
Git history. It requires an owner decision before the first push.

## Current Git state

Audit timestamp: `2026-10-30T18:53:20Z`. Workspace root
`/home/iffi/dev-workspace` is **not** a Git repository and has no `.git` parent.

### Control-plane repository

| Item | Value |
| --- | --- |
| Path | `/home/iffi/dev-workspace/control-plane` |
| Branch | `phase1/control-plane` (local, not published, no upstream) |
| HEAD | `aef3e1ed45c66ef2382e5cc794665d1720f7ad77` |
| Other local branch | `agent/shadow-archives/qdn-grouped-publish-outcome-20261001` (same SHA, tracks origin) |
| Remote | `origin` = `https://github.com/iffinland/ai-development-orchestration.git` |
| Fetch refspec | Narrow: only `agent/shadow-archives/...` is fetched |
| Tracked files | 67 |
| Unstaged modified (pre-existing) | 18 |
| Staged | 0 |
| Untracked before this task | 8 files (4 porcelain top-level entries) |
| Untracked after this task | 10 files (8 pre-existing + 2 new task files) |
| Linked worktrees | Only the primary checkout |
| Stash | None recorded by the prior audit; not modified |
| Git hooks | No non-sample hooks |
| Remote name collision | None in this repository; `origin` is not the new target |

Pre-existing modified tracked files (unchanged by this task): `AGENTS.md`,
`ENVIRONMENT.md`, `GIT-AND-HANDOFF.md`, `PROJECT-REGISTRY.md`, `README.md`,
`WORKFLOW.md`, `roles/DEEPSEEK.md`, `skills/README.md`, eight
`skills/qortal/*/SKILL.md`, `templates/REPORT.md`, `templates/TASK.md`.

Pre-existing untracked paths: `adapters/` (2 files), `docs/` (2 files),
`platforms/` (3 files), `tools/validate_control_plane.py` (1 file).

Added by this task: `.gitignore` and `tools/validate_security.py` under
`control-plane/`, plus `docs/.gitignore` in the central tree.

### Central documentation tree

| Item | Value |
| --- | --- |
| Path | `/home/iffi/dev-workspace/docs` |
| Git repository | No (no `.git` in path or parent) |
| Size / files | 236K before this task; 16 files after adding `.gitignore` |
| Project roots | `Blogs`, `nodefm-station`, `Qortium-United-Community`, `shadow-archives-webportal-QORTAL`, `Qortal-Video-Bridge`, `wordpress-to-qortal`, `workspace` |
| Category directories present | `workspace/audit`, `workspace/reports`, `Blogs/validation` (only where artifacts exist) |
| Nested Git repositories | None under `docs/` |
| Symlinks | None under `docs/` or `control-plane/` |
| Sensitive files | None matching `.env`, key, keystore, wallet, credential or secret filename patterns |

### Target remotes (read-only observation)

`GIT_TERMINAL_PROMPT=0 git ls-remote --heads` returned exit 0 with no heads for
both `https://github.com/iffinland/dev-control-plane.git` and
`https://github.com/iffinland/dev-workspace-docs.git`. They are reachable and
appear empty. Visibility and write permission were **not** independently
verified and no write was attempted.

Git credential/tooling state observed without printing values: `gh` is not
installed; global `user.name` and `user.email` are now configured (values not
repeated here); global `credential.helper` is `cache --timeout=43200`; no
`GITHUB_TOKEN`, `GH_TOKEN`, `GIT_ASKPASS` or `SSH_AUTH_SOCK` is set; `~/.ssh`
contains only `authorized_keys` and no client key. Write authentication remains
**unverified**.

## Sensitive-data findings (redacted)

No credentials, API keys, tokens, private keys, wallet files, keystore files or
credential-bearing URLs were found in the in-scope trees. The only confirmed
sensitive infrastructure literal is one historical public IPv4 address.

| ID | Path | Line | Category | Status | Proposed action |
| --- | --- | --- | --- | --- | --- |
| S1 | `control-plane/reports/2026-09-13-phase-2-qortal-node-b-tunnel.md` | 12 | Public VPS IPv4 address (redacted) | Historical prose; also present in existing commit `c3c745d` | Owner decision: accept for a private repository with preserved history, or approve a sanitizing edit/replacement before first push |
| S2 | Same historical report and `control-plane/reports/2026-09-13-phase-2-reference-sync-and-tunnels.md` | multiple | SSH aliases `qortium-vps`, `qortal-node-a`, `qortal-node-b`; remote path `/home/iffiolen/qortal` | Historical provenance; no credential values | Low risk. Optionally replace alias/host detail with role descriptions, only with owner approval |
| S3 | `control-plane/reports/2026-09-13-git-baseline.json` and several `tasks/` files | multiple | Repository names and SSH-form GitHub remote URLs (`git@github.com:...`) | No credentials; some repository names are private-sounding | Low risk. Keep as provenance or redact repository names if the new repository visibility changes |
| S4 | Active skills and task artifacts | many | Loopback addresses (`127.0.0.1`) and local API ports | Documentation-only, not externally reachable | No action; the validator ignores loopback |

### Confirmed literal address

The only public literal is the single address in S1. It is a historical VPS/node
host recorded in a Phase 2 report. It is **not** an active operational example
in current guidance: current control-plane and central documentation use
loopback addresses or generic endpoint descriptions.

### Masking proposal (not applied)

If the owner wants the historical report sanitized in the working tree:

```text
replace:  <the literal public host address in S1>
with:     ***.***.***.***
```

A role-based replacement such as `<node-b-host>` would preserve readability.
This edit is **not** applied automatically because the file is a historical
report and the task prohibits automatic historical rewrites.

Important limitation: masking the working tree does **not** remove the address
from existing Git objects. It remains in commit `c3c745d` and every descendant
history. Removing it from history requires a rewrite or a fresh-history import,
both of which require explicit owner authorization.

### Credential and secret scan result

Patterns checked across `control-plane/` and `docs/` (`.git` excluded): private
key blocks, GitHub/AWS/Slack/OpenAI-style token formats, credential-bearing URL
userinfo, assigned-secret key/value patterns, sensitive filenames, and
wallet/keystore/mnemonic indicators. No values were printed to the terminal or
written to this report. Result: no confirmed credentials or secret material.

The docs tree contains only policy, audit, validation and environment prose.
`docs/workspace/reports/DEVELOPMENT-ENVIRONMENT.md` contains the placeholder
`sinu@email.tld`, which is not a real secret.

## Files changed

| Path | Change | Purpose |
| --- | --- | --- |
| `/home/iffi/dev-workspace/control-plane/.gitignore` | New | Prevent accidental commit of local secrets, keys, logs, caches and scratch/backup files without hiding reports, tasks, skills or governance docs |
| `/home/iffi/dev-workspace/control-plane/tools/validate_security.py` | New | Standalone security validator for exposed infrastructure IPs and high-confidence secret patterns |
| `/home/iffi/dev-workspace/docs/.gitignore` | New | Equivalent protection for the future `dev-workspace-docs` repository |
| `/home/iffi/dev-workspace/docs/workspace/audit/GIT-SECURITY-MIGRATION-PREPARATION.md` | New | This report |

No existing tracked file was modified. No existing documentation file was
moved, renamed or rewritten. The pre-existing 18 modified and 8 untracked
control-plane entries are preserved exactly as found; the only status delta is
the two new control-plane files.

SHA-256 of the new non-report files before this report was written:

```text
fcc6ec234150620968ac23f8bab7ff4c88e86e8692d05ee42e2d38301a989578  control-plane/.gitignore
f22e56a8e9b7a612230e770f018f103786484547638216a7f7b04c682e7bc0a1  control-plane/tools/validate_security.py
e0f9135971bd6b758889db84b1d81dea7eef53fa33fc219403bd9054fb0897c4  docs/.gitignore
```

### `.gitignore` safety confirmation

- Control-plane ignored file list is empty (`git ls-files --others --ignored
  --exclude-standard`), so no current file is hidden.
- `git check-ignore` confirms the following remain tracked/committable:
  `AGENTS.md`, historical reports, task artifacts, report-storage policy,
  skills index, validator scripts, templates, platform packs and adapters.
- The docs rules were tested in a disposable temporary repository; existing
  report and README paths were not ignored.
- The rules deliberately contain no `reports/`, `tasks/`, `docs/`, `skills/`,
  `platforms/`, `adapters/`, `roles/`, `templates/` or `tools/` exclusion.

### Security validator behavior

`control-plane/tools/validate_security.py`:

- uses only the Python standard library;
- scans tracked and untracked non-ignored files via
  `git ls-files -co --exclude-standard` in a Git repository, and walks other
  directories;
- detects public IPv4/IPv6 addresses and, with `--include-private`, RFC1918/ULA
  addresses;
- ignores loopback, unspecified, link-local, multicast/reserved, RFC 5737 /
  RFC 3849 documentation ranges, common public resolvers and obvious
  placeholders such as `1.2.3.4`;
- detects private-key blocks, GitHub/AWS/Slack/OpenAI-style tokens,
  credential-bearing URLs, assigned secrets and sensitive filenames;
- masks all reported values;
- supports `--self-test`;
- exits `0` when clean, `1` when findings exist and `2` on usage errors.

It is intentionally **not** wired into the existing governance validator or a
pre-commit hook yet. Adding it to enforcement is a proposal, not a change made
here.

## Validator and check results

| Check | Result | Classification |
| --- | --- | --- |
| `python3 tools/validate_control_plane.py` | PASS: 38 active Markdown files, 85 relative links, 20 required files | Pre-existing validator; no failure |
| `python3 tools/validate_skills.py` | PASS: 15 active skills indexed exactly once | Pre-existing validator; no failure |
| `python3 tools/validate_security.py --self-test` | PASS | New validator self-test |
| `python3 -m py_compile tools/validate_security.py` | PASS | New file syntax check |
| `python3 tools/validate_security.py` (control-plane) | FAIL: 1 finding, S1 historical public IPv4 | **Pre-existing**; not introduced by this task |
| `python3 tools/validate_security.py /home/iffi/dev-workspace/docs` | PASS: 16 files scanned | New check; clean |
| `git diff --check` (control-plane) | PASS | No whitespace errors in tracked changes |
| `.gitignore` essential-path check | PASS | No essential path ignored |
| `git ls-remote --heads` on both new repositories | Exit 0, no heads | Read-only; empty and reachable |

Existing failures are exactly the pre-existing S1 security finding. The
existing control-plane and skills validators did not fail. No new validator
failure was introduced by the added `.gitignore` or validator files. The new
validator correctly reports the historical literal because that file is intended
to be committed if full history/provenance is preserved.

## Repository migration preparation

### Control-plane: recommended safest method

**Recommendation: preserve the existing Git history, add the new repository as a
second remote, commit the current work on a dedicated migration branch, then push
that branch.** This preserves all current work, avoids re-cloning, avoids a
destructive in-place orphan reset, and keeps the old `origin` untouched for
provenance.

Reasons:

1. The entire Phase 1/1.6 rewrite is uncommitted; a machine loss would lose it.
   Committing it is the safest preservation step.
2. The existing history is small (one 219 KiB pack) and already contains the
   historical report. A private repository does not republish that history to a
   new public audience.
3. `origin` currently points at a shared orchestration repository with a narrow
   fetch refspec. Adding a separate remote avoids accidentally pushing to the
   old repository.
4. Rewriting or re-rooting history is not required by the task and would
   conflict with the "preserve appropriate Git history" objective.

Alternative only if the owner requires a sanitized history: perform a fresh
import into a **separate staging copy** (do not use `git switch --orphan` in the
live checkout; it removes tracked files from the working tree). See Option B
below.

### Central documentation: recommended initialization

Initialize `/home/iffi/dev-workspace/docs` as a new Git repository in place,
because it is currently outside all Git repositories, has no history to
preserve, contains no nested `.git` directories and no symlinks. Add
`dev-workspace-docs` as its `origin` only in the authorized phase, then push the
initial commit.

### Migration conflict and preservation checks

| Check | Result |
| --- | --- |
| Nested Git repositories under `docs/` | None |
| Nested Git repositories under `control-plane/` | Only the expected control-plane `.git` |
| Symlinks under `docs/` or `control-plane/` | None |
| Sensitive files (`env`, keys, wallets, keystores) | None found |
| Existing worktree preservation | 18 pre-existing modified files unchanged; 8 pre-existing untracked files unchanged |
| Docs content preservation | All existing docs files unchanged; only `.gitignore` added |
| File overlap between the two future repositories | None: control-plane governance docs stay in `dev-control-plane`; central report tree stays in `dev-workspace-docs` |
| Path portability | Reports contain absolute `/home/iffi/dev-workspace/...` paths; not secret, but not portable |
| History conflict | S1 public IP is in committed history; masking the worktree does not remove it |

## Recommended migration sequence

This sequence is a proposal. Nothing below was executed.

1. **Owner decisions.** Resolve S1 (accept for private history or approve
   sanitization), choose full-history vs fresh-history migration, choose the
   default branch name for `dev-control-plane`, and confirm write credential.
2. **Freeze and verify control-plane state.** Record `git status`, HEAD and
   hashes of the new files. Re-run all three validators. Confirm no new secret
   appeared.
3. **Commit current control-plane work locally** on a migration branch. Do not
   push yet.
4. **Add `dev-control-plane` as a second remote.** Keep `origin` unchanged.
5. **Push the migration branch** and verify the remote SHA with `git ls-remote`.
6. **Publish the chosen default branch** (`main` or the migration branch) only
   after owner review.
7. **Initialize `docs/` as `dev-workspace-docs`**, commit, add `origin`, push,
   and verify the remote SHA.
8. **Post-push validation.** Clone both repositories into disposable locations,
   run the security validator there, confirm the report tree and control-plane
   structure are intact, and run the two existing validators from the fresh
   control-plane clone.
9. **Optional later cleanup.** Only with separate authorization, rename the old
   remote to `legacy-orchestration` or repoint `origin`; do not combine this with
   the first push.

## Exact proposed Git operations for a later authorized phase

The commands below are proposals only. Do not run them until the owner approves
the specific phase and credential.

### Option A (recommended): preserve history, add a second remote

```bash
cd /home/iffi/dev-workspace/control-plane

# Preflight: record state and validate
git status --porcelain=v1 -uall
git rev-parse HEAD
git remote -v
python3 tools/validate_control_plane.py
python3 tools/validate_skills.py
python3 tools/validate_security.py   # expected: S1 historical finding

# Add the new target without touching origin
git remote add dev-control-plane https://github.com/iffinland/dev-control-plane.git

# Create a reviewable migration branch and commit the preserved work
git switch -c migration/control-plane-phase1-20261030 phase1/control-plane
git add -A
git diff --cached --check
git diff --cached --stat
git commit -m "chore(control-plane): checkpoint phase 1 canonical workspace"

# Push the migration branch first (does not publish main yet)
git push -u dev-control-plane migration/control-plane-phase1-20261030
git ls-remote --heads dev-control-plane
```

After owner review, publish the default branch in one of these ways:

```bash
# Option A1: push the reviewed branch as remote main
git push dev-control-plane migration/control-plane-phase1-20261030:main

# Option A2: create a local main and push it
git switch -c main migration/control-plane-phase1-20261030
git push -u dev-control-plane main
```

If the repository has branch protection or a required PR, open the PR instead of
pushing `main` directly.

### Option B (only if sanitized/fresh history is required)

Do not run `git switch --orphan` in the live checkout: in this environment it
removes tracked files from the working tree. Use a separate staging copy.

```bash
mkdir -p /home/iffi/dev-workspace/staging
cp -a /home/iffi/dev-workspace/control-plane/. /home/iffi/dev-workspace/staging/control-plane-fresh/
rm -rf /home/iffi/dev-workspace/staging/control-plane-fresh/.git
cd /home/iffi/dev-workspace/staging/control-plane-fresh

# Apply owner-approved S1 masking here, before the first commit.

git init -b main
git add -A
git diff --cached --check
git commit -m "chore(control-plane): initial canonical workspace (fresh history)"
git remote add origin https://github.com/iffinland/dev-control-plane.git
git push -u origin main
git ls-remote --heads origin
```

Option B discards the historical commit graph and any S1 masking must be
explicitly authorized because it edits a historical report.

### Central documentation initialization

```bash
cd /home/iffi/dev-workspace/docs

# Pre-init checks: must show no `.git` and no symlinks
find . -name .git -o -type l
python3 /home/iffi/dev-workspace/control-plane/tools/validate_security.py /home/iffi/dev-workspace/docs

git init -b main
git add -A
git diff --cached --check
git diff --cached --stat
git commit -m "docs: initialize central workspace documentation"
git remote add origin https://github.com/iffinland/dev-workspace-docs.git
git push -u origin main
git ls-remote --heads origin
```

### Post-push verification (read-only)

```bash
git ls-remote --heads https://github.com/iffinland/dev-control-plane.git
git ls-remote --heads https://github.com/iffinland/dev-workspace-docs.git

# In disposable clones:
python3 tools/validate_control_plane.py
python3 tools/validate_skills.py
python3 tools/validate_security.py
python3 /path/to/validate_security.py /path/to/cloned-docs
```

## Risks and unresolved decisions

1. **S1 public infrastructure literal is already in history.** A private target
   repository reduces new exposure, but the old `origin` history is unchanged.
   The owner must decide whether to accept this for `dev-control-plane` or
   authorize sanitization/history re-rooting.
2. **Work is still uncommitted.** Every local change remains at risk until the
   owner authorizes the first commit. This task deliberately did not commit.
3. **Write credentials are unverified.** A credential helper cache is configured
   and global identity is now set, but no push was attempted and `gh` is absent.
   The owner must verify write access before the push phase.
4. **Default branch policy is undecided.** `main` versus the migration branch,
   branch protection, PR requirement and signed-commit policy need owner choice.
5. **Full history versus sanitized history is undecided.** Preserving history is
   safer for provenance but retains S1; fresh history removes S1 only if the
   historical report is also edited.
6. **Historical task/report material may contain private repository names and
   old workstation paths.** These are not credentials, but the owner may want a
   disclosure review before pushing to a differently scoped repository.
7. **Root workspace material is out of scope.** `/home/iffi/dev-workspace/ENVIRONMENT-AUDIT.md`,
   `scripts/env.sh`, `logs/`, `tools/` and `backups/` are outside both target
   repositories. Decide later whether any belong in a repository; do not include
   `backups/` or `scripts/env.sh` without a separate secret review.
8. **Validator enforcement is not wired.** The new validator is standalone; CI
   or pre-commit integration is a separate governance change.
9. **Absolute paths reduce portability.** Central docs and control-plane use
   `/home/iffi/dev-workspace/...`; this is environment-specific but not secret.
10. **Two-repository boundary must remain intentional.** Do not accidentally
    commit control-plane governance docs into the docs repository or central
    reports into the control-plane repository.

## Checks not executed and why

- No commit, push, fetch, pull, remote add/rename, tag, merge, branch creation or
  history rewrite.
- No `git init` in `/home/iffi/dev-workspace/docs`.
- No GitHub visibility, permission, issue, PR or release change.
- No write-access test against either target repository.
- No inspection of `backups/`, `scripts/env.sh`, `logs/` or runtime `tools/`
  contents; they are outside the requested control-plane/docs scope.
- No contact with Qortium Core, Qortal Core, a node, wallet, API key or runtime
  service.
- No automatic modification of historical reports, task artifacts or Git
  history.
- No credential values were printed, copied or written to this report.

## Adversarial self-audit

- Confirmed the workspace root and `docs/` are outside Git, and that `docs/`
  contains no nested repository or symlink before proposing initialization.
- Confirmed the control-plane has no non-sample hooks and no credential-bearing
  remote URL.
- Checked the only public IPv4 candidate against loopback, private, link-local,
  reserved, documentation and placeholder ranges; exactly one true finding
  remains, in a historical report.
- Ran the new validator's self-test against clean, public-IP, private-IP,
  token and credential-URL fixtures. The self-test initially flagged its own
  literal fixtures; the fixtures were changed to runtime-constructed values so
  the validator scans itself cleanly.
- Verified `.gitignore` does not hide reports, tasks, docs, skills, platforms,
  adapters, roles, templates or validator tools, using both `git check-ignore`
  and a disposable docs repository.
- Verified the pre-existing working-tree state is unchanged except for the two
  new control-plane files and the one new docs file; no tracked content was
  edited.
- Confirmed the proposed fresh-history method avoids the destructive behavior of
  `git switch --orphan` observed in a disposable test repository.
- Redaction check: this report contains no raw public IP, token, password,
  private key, email address or credential value. Values in validator output are
  masked.

## External actions (commit/push/tag/release/deploy/QDN write/transaction)

None. The only network operations were read-only `git ls-remote --heads`
requests to the two empty target repositories. No remote state was changed.

## Remaining risks and follow-up

- Owner must decide on S1 and the full-history versus fresh-history strategy.
- Owner must authorize the first commit and push, and confirm the write
  credential and default branch.
- After authorization, run the proposed post-push clones and validators.
- Consider adding `tools/validate_security.py` to CI or a pre-commit hook in a
  separate governance task.
- Keep `backups/`, `scripts/env.sh`, logs and runtime tools out of both
  repositories unless separately reviewed.

## Report saved

- Canonical absolute path: `/home/iffi/dev-workspace/docs/workspace/audit/GIT-SECURITY-MIGRATION-PREPARATION.md`
- Category: `audit`
- SHA-256: computed after write, not embedded; the report cannot contain its own final hash.
- Validation: existing control-plane and skills validators pass; new security
  validator self-test passes; control-plane scan reports only the pre-existing
  S1 historical finding; docs scan passes; worktree changes are limited to the
  three new non-report files and this report.
