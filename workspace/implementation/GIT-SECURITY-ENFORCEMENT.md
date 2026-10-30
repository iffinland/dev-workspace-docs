# Git security enforcement and commit preparation — workspace/implementation

Executing agent (registered role of the agent that actually produced this work): DeepSeek
Report/handoff writer (if different from the executing agent): DeepSeek (same)
Executing-agent evidence (how the real executor was established): Work was executed in this session by the model identified in-session as `deepseek/deepseek-flash`; the control-plane contract, workflow, handoff rules, report-storage policy and `roles/DEEPSEEK.md` were read and followed. The CLI/orchestration profile is not used as identity.
Report type / category (audit, reports, research, implementation, validation, handoffs): implementation
Project ID / central report root: cross-project / control plane → `/home/iffi/dev-workspace/docs/workspace/`
Exact application repository / branch / SHA: control-plane `phase1/control-plane` @ `aef3e1ed45c66ef2382e5cc794665d1720f7ad77`; central docs: N/A (not a Git repository yet)
Canonical report path `docs/<project>/<category>/<dated-name>.md` / SHA-256 / authorized remote evidence: `/home/iffi/dev-workspace/docs/workspace/implementation/GIT-SECURITY-ENFORCEMENT.md`; SHA-256 computed after write (not embedded); no remote evidence (local-only implementation)

## Objective and exit criterion

Integrate the existing security validator into the Git workflow so that new
sensitive-data leaks are blocked before commits and pushes, prepare the
control-plane and central documentation for their first reviewed commits, and
do so without committing, pushing, changing remotes or branches, initializing
the docs repository, or rewriting history.

Exit criterion: shared validator and shared hook driver implemented;
repository-local enforcement prepared for both target repositories; pre-commit
checks exact staged content; pre-push checks outgoing commits including deleted
blobs; historical finding has a narrowly scoped fingerprint exception; all
required behavioral tests pass in disposable repositories; existing validators
and whitespace checks pass; thematic commit plans are recorded; no prohibited
action is performed.

Result: **PASS for the authorized local scope.** Control-plane enforcement is
active. Docs enforcement is prepared but cannot be activated until the owner
authorizes `git init` (explicitly prohibited in this task).

## Implementation architecture

The security logic is implemented once and shared:

```text
control-plane/tools/validate_security.py   shared scanner + exception engine
control-plane/tools/git_hooks.py           shared pre-commit/pre-push driver
control-plane/tools/security-exceptions.json
control-plane/tools/install_git_hooks.sh   local core.hooksPath installer
control-plane/tools/test_git_hooks.py      disposable-repository test suite

control-plane/.githooks/pre-commit         thin repository wrapper
control-plane/.githooks/pre-push           thin repository wrapper
docs/.githooks/pre-commit                  thin repository wrapper
docs/.githooks/pre-push                    thin repository wrapper
docs/.security-exceptions.json             empty docs exception policy
```

The wrappers contain only path discovery and an `exec` of the shared driver.
They do not duplicate scanning logic. Each repository uses
`core.hooksPath=.githooks`, so the hook files are version-controlled rather
than copied into `.git/hooks`.

### Pre-commit behavior

1. Reads the repository index with `git ls-files --stage -z`.
2. Selects added/copied/modified/renamed paths with
   `git diff --cached --name-only -z --diff-filter=ACMR`.
3. Reads the exact staged blob OID, not the working-tree file.
4. Scans text content and the staged path name with the shared validator.
5. Fails if the index cannot be read, an entry is unmerged, a path is missing,
   or the validator cannot run.

This means an unstaged secret in the working tree does not block a clean staged
version, while a staged secret blocks even if the working tree was cleaned
afterward.

### Pre-push behavior

1. Reads the ref update lines from stdin.
2. Resolves outgoing commits with `git rev-list` against the remote base when
   that object exists locally. If the remote base is absent, it conservatively
   scans all commits reachable from the local tip.
3. For every outgoing commit, determines changed paths with
   `git diff-tree --root -r -m --no-commit-id --name-only -z`.
4. Reads the blob at `<commit>:<path>` for each changed path. This includes
   blobs that no longer exist at the current tip.
5. Scans every changed blob and every outgoing commit message.
6. Deduplicates commits, paths and blobs across refs.
7. Fails if ref parsing, Git plumbing or validation fails.

A secret added in one outgoing commit and deleted in a later outgoing commit is
therefore still scanned at the commit where it existed.

### Detection policy

Default behavior now matches the required policy:

- public IPv4/IPv6 infrastructure addresses are blocked;
- private RFC1918 and IPv6 ULA addresses are blocked by default;
- loopback, unspecified, link-local, multicast/reserved, RFC 5737/3849
  documentation ranges, common public resolvers and obvious placeholders remain
  allowed;
- private-key blocks, GitHub/AWS/Slack/OpenAI-style tokens, credential-bearing
  URLs, assigned secrets and sensitive filenames are blocked;
- output is always masked; no raw secret or full infrastructure address is
  printed;
- `--no-private` exists only for an explicitly approved documentation audit and
  is not used by the hooks.

## Exact files created or modified

| Path | Status | Purpose |
| --- | --- | --- |
| `control-plane/tools/validate_security.py` | Modified/rewritten | Shared scanner, private-address default, exception engine, masked output, self-test |
| `control-plane/tools/git_hooks.py` | Created | Shared pre-commit/pre-push driver using Git plumbing |
| `control-plane/tools/security-exceptions.json` | Created | Narrow historical exception configuration |
| `control-plane/tools/test_git_hooks.py` | Created | 16 disposable-repository enforcement tests |
| `control-plane/tools/install_git_hooks.sh` | Created | Installs `core.hooksPath=.githooks` in an authorized repository |
| `control-plane/.githooks/pre-commit` | Created | Thin control-plane wrapper |
| `control-plane/.githooks/pre-push` | Created | Thin control-plane wrapper |
| `docs/.githooks/pre-commit` | Created | Thin docs wrapper, prepared for activation |
| `docs/.githooks/pre-push` | Created | Thin docs wrapper, prepared for activation |
| `docs/.security-exceptions.json` | Created | Empty docs exception policy |
| `docs/workspace/implementation/GIT-SECURITY-ENFORCEMENT.md` | Created | This report |

Not modified by this task: existing governance documents, historical reports,
task artifacts, skills, templates, `control-plane/.gitignore`,
`docs/.gitignore`, Git history, remotes, branches or GitHub settings.

SHA-256 of the new implementation files (excluding this report, whose final hash
is computed after write):

```text
1933ece46924fd93056debcb086d9ba3bfcfbc41c587d941ddebb7f59039fa12  control-plane/.githooks/pre-commit
1933ece46924fd93056debcb086d9ba3bfcfbc41c587d941ddebb7f59039fa12  control-plane/.githooks/pre-push
69963aaa04475d2763f14d0c4f77016433f14c498f18d4caf4088756fd3d1991  control-plane/tools/git_hooks.py
cf25069fe35078ab0dcce53201ec9809889937589f1d2290ca5002c6f6a3516e  control-plane/tools/security-exceptions.json
1daf0865530e1a1a3da5634da1d2a583c96c42a6db07b8cdd7c0a8c802070acf  control-plane/tools/test_git_hooks.py
ccc66b8f49d379aa66e3c23551f1683ce691368a62d98fef98da6fffb8bce326  control-plane/tools/install_git_hooks.sh
77ea70acb7aa7002b5057308f46abb9a1c5e71107ef0243348385726fd161e80  control-plane/tools/validate_security.py
1933ece46924fd93056debcb086d9ba3bfcfbc41c587d941ddebb7f59039fa12  docs/.githooks/pre-commit
1933ece46924fd93056debcb086d9ba3bfcfbc41c587d941ddebb7f59039fa12  docs/.githooks/pre-push
1737ab15129a4c552c165b9809a2c6961508b263154ab3ad61113f609b7d360b  docs/.security-exceptions.json
```

The hook wrapper files are intentionally identical; the hook name is derived
from the invoked filename.

## Hook installation and activation status

| Repository | Hook files present | `core.hooksPath` | Activation |
| --- | --- | --- | --- |
| `control-plane` | Yes, `.githooks/pre-commit` and `.githooks/pre-push` | `.githooks` (local Git config) | **Active** |
| `docs` | Yes, `.githooks/pre-commit` and `.githooks/pre-push` | Not applicable (not a Git repository) | **Prepared, not active** until authorized `git init` and config |

The control-plane activation was verified with a direct wrapper smoke test: no
staged content, clean pass. The installer is idempotent:

```bash
control-plane/tools/install_git_hooks.sh /home/iffi/dev-workspace/control-plane
```

Docs activation after an authorized initialization (not executed here):

```bash
cd /home/iffi/dev-workspace/docs
git init -b main
/home/iffi/dev-workspace/control-plane/tools/install_git_hooks.sh /home/iffi/dev-workspace/docs
```

If the docs repository is later cloned outside this workspace sibling layout,
set `CONTROL_PLANE_ROOT` to the shared control-plane checkout or vendor the
shared tooling. Otherwise the wrappers fail closed.

## Historical exception implementation

The owner explicitly accepted the single pre-existing public IPv4 finding in:

```text
control-plane/reports/2026-09-13-phase-2-qortal-node-b-tunnel.md
```

The exception is stored in
`control-plane/tools/security-exceptions.json` and is keyed by:

- repository-relative path;
- finding kind (`public IPv4 address`);
- exact line number;
- SHA-256 of the complete normalized line.

The actual address is **not** present in the exception file or this report. The
fingerprint covers the whole line, not only the address, so it is not a
standalone address hash.

Scope guarantees:

- Only the exact path, line, kind and line content match; the file is still
  scanned completely.
- Any additional IP or credential anywhere else in that file is reported.
- The same historical value in any other file is reported because the path does
  not match.
- The same value on another line of the same file does not match.
- Editing or reformatting the excepted line makes the exception stop matching;
  the hook fails closed and requires an owner-approved exception update.
- No exception can suppress a token, and no wildcard or whole-file exemption is
  supported.

## Validation test results

`python3 control-plane/tools/test_git_hooks.py` runs entirely in disposable
temporary repositories. It does not create commits in the canonical
repositories. All 16 cases passed:

| # | Case | Expected | Result |
| --- | --- | --- | --- |
| 1 | Clean staged changes | PASS | PASS |
| 2 | New public IP | BLOCK | PASS |
| 3 | New private infrastructure IP | BLOCK | PASS |
| 4 | New token / credential URL | BLOCK | PASS |
| 5 | Unstaged working-tree secret only | PASS | PASS |
| 6 | Same secret staged | BLOCK | PASS |
| 7 | Historical exception unchanged (pre-commit) | PASS | PASS |
| 8 | Additional IP in historical file | BLOCK | PASS |
| 9 | Same historical IP in another file | BLOCK | PASS |
| 10 | Secret added then deleted in outgoing commits | BLOCK at pre-push | PASS |
| 11 | Clean outgoing commits | PASS at pre-push | PASS |
| 12 | Historical exception on full-history pre-push | PASS | PASS |
| 13 | Historical IP in another file at pre-push | BLOCK | PASS |
| 14 | Credential in outgoing commit message | BLOCK at pre-push | PASS |
| 15 | Validator unavailable | BLOCK | PASS |
| 16 | Validator broken at import time | BLOCK | PASS |

Other checks:

| Check | Result |
| --- | --- |
| `python3 tools/validate_control_plane.py` | PASS: 38 active Markdown files, 85 links, 20 required files |
| `python3 tools/validate_skills.py` | PASS: 15 active skills indexed exactly once |
| `python3 tools/validate_security.py --self-test` | PASS |
| `python3 tools/validate_security.py` (control-plane) | PASS: exception applied; 82 files scanned |
| `python3 tools/validate_security.py /home/iffi/dev-workspace/docs` | PASS: 20 files scanned before this report |
| `python3 -m py_compile` on new Python files | PASS |
| `bash -n` on all four hook wrappers and installer | PASS |
| `git diff --check` / `git diff --cached --check` | PASS |
| `.gitignore` check for new files | New hook/validator/exception files are not ignored |
| Control-plane hook smoke test | PASS: no staged changes, clean exit |
| Raw historical address in new files | None found |

## Current Git status

Control-plane:

- Branch: `phase1/control-plane`
- HEAD: `aef3e1ed45c66ef2382e5cc794665d1720f7ad77`
- Remote: `origin` = `https://github.com/iffinland/ai-development-orchestration.git`
- Staged: none
- Modified tracked: 18 (pre-existing Phase 1/1.6 work; unchanged by this task)
- Untracked: 16 (8 prior Phase 1 files, `.gitignore`, `tools/validate_control_plane.py`, and 6 new security files)
- Linked worktrees: primary checkout only
- Local config: `core.hooksPath=.githooks` (local only, not a tracked file)

Central docs:

- Not a Git repository; no `.git` initialized.
- 20 files before this report; 21 after this report is added.
- No nested Git repository, no symlink.
- New docs enforcement files are untracked local files until the authorized
  initial commit.

No commit, no push, no remote change, no branch change and no docs
initialization occurred.

## Proposed thematic commit boundaries

Files are **not staged**. The control-plane changes should be reviewed and
committed in three thematic commits.

### Commit 1 — Governance and workspace structure

- Modified: `AGENTS.md`, `ENVIRONMENT.md`, `GIT-AND-HANDOFF.md`,
  `PROJECT-REGISTRY.md`, `README.md`, `WORKFLOW.md`, `roles/DEEPSEEK.md`,
  `skills/README.md`, the eight modified `skills/qortal/*/SKILL.md` files,
  `templates/REPORT.md`, `templates/TASK.md`
- New: `adapters/deepseek/README.md`, `adapters/opencode/README.md`,
  `platforms/README.md`, `platforms/qortal/README.md`,
  `platforms/qortium/README.md`, `tools/validate_control_plane.py`

### Commit 2 — Documentation and report-storage migration

- New: `docs/REPORT-STORAGE-POLICY.md`
- New: `docs/migration/2026-10-30-phase1-control-plane.md`

### Commit 3 — Security validator and Git enforcement

- New: `.gitignore`
- New: `.githooks/pre-commit`, `.githooks/pre-push`
- New: `tools/validate_security.py`, `tools/git_hooks.py`,
  `tools/security-exceptions.json`, `tools/test_git_hooks.py`,
  `tools/install_git_hooks.sh`

No pre-existing historical report or task artifact is modified. No file needs
to be excluded from these commits. Do not commit `__pycache__/`, local Git
config, `backups/`, `logs/`, runtime `tools/` or any owner-local environment
file; the ignore rules and the current untracked list already exclude the
first two from version control in practice.

### Separate initial commit plan for `dev-workspace-docs`

After explicit authorization to initialize docs:

1. Verify no `.git` exists and no nested repository/symlink is present.
2. `git init -b main`.
3. Install the hooks with `control-plane/tools/install_git_hooks.sh` so
   `core.hooksPath=.githooks` is active before the first commit.
4. Stage all intended docs files, including `.gitignore`,
   `docs/.security-exceptions.json`, both `.githooks` wrappers, all project
   READMEs, all audits/reports/validation artifacts and this implementation
   report.
5. Run the active pre-commit hook and the standalone security scan against the
   docs tree.
6. Commit as one initial documentation commit, then review before any push.

Do not initialize or commit docs in this phase.

## Risks and limitations

1. **Local hooks are bypassable.** `git commit --no-verify` and
   `git push --no-verify` skip them. Removing `core.hooksPath` also disables
   them. Enforcement must be paired with remote-side branch protection or CI
   for a trusted gate.
2. **Docs enforcement is not active yet.** The docs directory is not a Git
   repository. The wrappers and config are prepared; activation requires the
   owner-authorized initialization step.
3. **Shared tooling dependency.** Docs wrappers depend on the control-plane
   checkout or `CONTROL_PLANE_ROOT`. A standalone docs clone must vendor or
   otherwise locate the shared driver, or hooks fail closed.
4. **Exception freshness.** The historical exception is tied to the exact line
   content. Line-ending conversion, reformatting or any edit to that line will
   cause a block until the owner approves an updated fingerprint.
5. **Binary/large files.** NUL-containing binary blobs are skipped; files over
   the 2 MiB scan limit cause the hook to fail closed for changed content.
   Documented text is the supported surface.
6. **Commit messages are scanned at pre-push but not at commit time.** A
   malformed message can still be created locally, but an outgoing push is
   blocked. Rewriting the local commit is then required.
7. **Remote-side state is unchanged.** No GitHub visibility, branch protection,
   required checks, secret scanning or permission was configured.
8. **The exception is a fingerprint, not a secret.** It stores a hash of the
   complete historical line, not the address. It is intentionally narrow, but
   the config is committed and visible to anyone who can read the repository.
9. **Pre-push conservative fallback.** If the remote base object is not present
   locally, all commits reachable from the local tip are scanned. This is
   fail-safe but can be slower on large histories.
10. **Work remains uncommitted.** The first reviewed commit still requires
    owner authorization.

## Owner decisions required

1. Approve the three-commit control-plane sequence and the exact file boundaries.
2. Authorize staging and the first local commits; no push was authorized here.
3. Choose control-plane default branch (`main` versus the migration branch) and
   whether a PR/branch protection is required before pushing.
4. Authorize docs `git init` and its single initial commit, then confirm hook
   activation with `core.hooksPath=.githooks`.
5. Confirm the historical exception fingerprint approach and that the
   historical report must remain byte-for-byte unchanged.
6. Decide whether remote-side CI/secret scanning or branch protection will
   supplement the bypassable local hooks.
7. Verify the push credential and write access before any push phase.
8. Decide whether the new security validator should also be referenced from
   `REPORT-STORAGE-POLICY.md` or a dedicated governance enforcement section.

## Checks not executed and why

- No commit, push, fetch, remote add/rename, branch creation, tag or history
  rewrite.
- No staging (`git add`) of canonical files.
- No `git init` in the docs directory.
- No change to GitHub visibility, permissions, branch protection or secrets.
- No modification of historical reports, task artifacts or runtime
  configuration.
- No contact with Qortium/Qortal services, wallets, keystores or API keys.
- No external dependencies added.

## Adversarial self-audit

- Verified the validator's exception match uses path, kind, line number and
  full-line SHA-256, so a whole-file bypass is not possible.
- Tested duplicate finding in the excepted file and the same value in another
  file; both block.
- Tested exact staged content by staging clean content and then writing a secret
  only in the working tree; the hook passed. Staging the secret then blocked.
- Tested a secret added and deleted across outgoing commits; pre-push blocked at
  the adding commit.
- Tested both historical exception paths: pre-commit and full-history pre-push.
- Tested missing and broken validator paths; the wrappers failed closed.
- Verified the test suite constructs sensitive fixtures at runtime, so the
  validator scans its own test file cleanly.
- Verified control-plane `core.hooksPath` is `.githooks` and the active wrapper
  runs clean with no staged content.
- Verified no existing tracked file was modified by this task; only new
  implementation files and this report were added.
- Redaction check: no raw public/private IP, token, credential URL, email or
  key value is present in this report or in the new implementation files.

## External actions (commit/push/tag/release/deploy/QDN write/transaction)

None. No Git write operation was performed. The only local Git configuration
change was `core.hooksPath=.githooks` in the control-plane repository, which is
a local hook-activation setting, not a commit, push, remote, branch or GitHub
change.

## Remaining risks and follow-up

- Owner review and authorization of the thematic commits.
- Docs initialization and hook activation remain pending.
- Remote-side enforcement should be added later if the owner wants protection
  against `--no-verify`.
- Potential future exception updates must remain exact and documented; never
  create a wildcard or whole-file exception.

## Report saved

- Canonical absolute path: `/home/iffi/dev-workspace/docs/workspace/implementation/GIT-SECURITY-ENFORCEMENT.md`
- Category: `implementation`
- SHA-256: computed after write, not embedded.
- Validation: all 16 disposable hook tests pass; existing control-plane and
  skills validators pass; security scans pass; whitespace checks pass; no
  canonical commit, push, staging or docs initialization was performed.
