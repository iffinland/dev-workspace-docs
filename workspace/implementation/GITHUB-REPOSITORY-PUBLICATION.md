# GitHub repository publication — workspace/implementation

Executing agent (registered role of the agent that actually produced this work): DeepSeek
Report/handoff writer (if different from the executing agent): DeepSeek (same)
Executing-agent evidence (how the real executor was established): Work was executed in this session by the model identified in-session as `deepseek/deepseek-flash`; the control-plane contract, workflow, handoff rules, report-storage policy and `roles/DEEPSEEK.md` were read and followed. The CLI/orchestration profile is not used as identity.
Report type / category (audit, reports, research, implementation, validation, handoffs): implementation
Project ID / central report root: cross-project / control plane → `/home/iffi/dev-workspace/docs/workspace/`
Exact application repository / branch / SHA: control-plane `main` @ `5b9f9e3813aac15a739ce85e027c46c8b572a2c6` (not pushed); central docs `main` @ `6589f2e625831b9b50c36aa7ee83f56f04437898` (not pushed)
Canonical report path `docs/<project>/<category>/<dated-name>.md` / SHA-256 / authorized remote evidence: `/home/iffi/dev-workspace/docs/workspace/implementation/GITHUB-REPOSITORY-PUBLICATION.md`; SHA-256 computed after write (not embedded); no remote evidence (publication blocked)

## Objective and exit criterion

Publish the reviewed local `control-plane` and `docs` repositories to the two
private empty GitHub repositories, after a mandatory pre-push security gate,
without force push, history rewrite, old-remote changes or secret disclosure.

Result: **PARTIAL**. Local preparation is complete and all content/security
validation passes, but publication was **not** performed. The active `pre-push`
wrapper rejects the two positional arguments Git supplies to a pre-push hook
(`<remote-name> <remote-url>`), so a normal push would exit non-zero before
transfer. Bypassing was prohibited; the task requires stopping when the pre-push
hook fails.

## Exact local and remote commit SHAs

| Repository | Local branch | Local SHA | Remote `main` |
| --- | --- | --- | --- |
| control-plane | `main` | `5b9f9e3813aac15a739ce85e027c46c8b572a2c6` | absent (not pushed) |
| docs | `main` | `6589f2e625831b9b50c36aa7ee83f56f04437898` | absent (not pushed) |

Preserved control-plane branches: `phase1/control-plane` at the same SHA and
`agent/shadow-archives/qdn-grouped-publish-outcome-20261001` at
`aef3e1ed45c66ef2382e5cc794665d1720f7ad77`.

Docs history is now two commits: `f564494` (initial) and `6589f2e`
(`docs: record verified local Git migration commits`).

## Repository URLs and remote configuration

Control-plane remotes:

```text
dev-control-plane  https://github.com/iffinland/dev-control-plane.git (fetch/push)
origin             https://github.com/iffinland/ai-development-orchestration.git (fetch/push, unchanged)
```

Docs remotes:

```text
origin             https://github.com/iffinland/dev-workspace-docs.git (fetch/push)
```

No token is embedded in either URL. No remote was renamed or removed.

## Authentication outcome (no credential values)

- `git credential fill` for `host=github.com` succeeded and contained both a
  username and a password field. Values were not printed, logged or written.
- `GITHUB_TOKEN`, `GH_TOKEN`, `GIT_ASKPASS` and `SSH_AUTH_SOCK` are unset.
- `gh` is not installed.
- Global `credential.helper` is `cache --timeout=43200`; only the helper name was
  read.
- Read-only `git ls-remote` succeeded on both targets. Write access was not
  exercised because the pre-push gate stopped publication before transfer.

## Pre-publication checks

| Check | Result |
| --- | --- |
| Control-plane preflight branch/HEAD/remote/hooks | Match expected exactly |
| Docs preflight branch/HEAD/untracked/remote/hooks | Match expected exactly |
| Both target repositories reachable | PASS (`git ls-remote` exit 0) |
| Both target repositories empty (all refs) | PASS (no refs) |
| Credentials available | PASS (no values disclosed) |
| Docs final commit (`docs: record verified local Git migration commits`) | `6589f2e`; pre-commit scanned 1 file; clean tree |
| Control-plane local `main` created at reviewed SHA | PASS |
| `dev-control-plane` remote added | PASS |
| Docs `origin` added | PASS |
| `validate_control_plane.py` | PASS: 38 Markdown files, 85 links, 20 required files |
| `validate_skills.py` | PASS: 15 skills indexed exactly once |
| `validate_security.py --self-test` | PASS |
| `validate_security.py` (control-plane) | PASS: 83 files scanned |
| `validate_security.py` (docs) | PASS: 22 files scanned |
| `test_git_hooks.py` (16 cases) | PASS: 16/16 |
| Pre-push hook files executable in both repos | PASS |
| Historical IP exception over full 15-commit control-plane history | PASS (driver direct scan: 125 blobs) |
| Docs full-history scan | PASS (driver direct scan: 2 commits, 22 blobs) |
| Targets still empty at gate | PASS |
| Actual pre-push wrapper invoked as Git invokes it | **FAIL (exit 2, argparse rejects `origin <url>`)** |

### Pre-push blocker detail

The wrappers end with:

```text
exec python3 "$control_plane/tools/git_hooks.py" "$(basename "$0")" "$@"
```

Git calls `pre-push` with two positional arguments (remote name and remote URL).
`git_hooks.py` accepts only the `hook` positional plus `--repo`/`--exceptions`,
so it exits 2 with `unrecognized arguments`. This is a functional hook defect,
not a security finding. Both generated scans pass when the shared driver is
invoked directly, which shows the security content and historical exception are
correct; the wrapper argument forwarding blocks a real push.

## Publication results

- Control-plane: **not published**.
- Docs: **not published**.
- No force push, no history rewrite, no tags, no other branches.
- No change to the old orchestration repository or any application repository.

## Working-tree status

- Control-plane: clean; four commits ahead of the preserved base
  (`30f37a0`, `3ea6177`, `5b9f9e3` plus the preserved base is the parent chain).
  Local `main` points at the reviewed security commit. Remote configured.
- Docs: clean before this report; only this report is untracked after write.
- Both repositories still have `core.hooksPath=.githooks`.

## Files changed in this task

- Docs commit `6589f2e`: added `workspace/implementation/GIT-LOCAL-COMMITS.md`.
- Control-plane: no new commit; created local branch `main`, added remote
  `dev-control-plane`.
- Docs: added remote `origin`.
- This report: new untracked file.

## Checks not executed and why

- No push was attempted after the pre-push wrapper failure, per the stop rule.
- No write-access test was performed; it requires an actual push.
- No branch protection, GitHub Actions, visibility or repository settings were
  changed; out of scope.
- No hook, validator or exception-policy file was modified; the failed gate must
  not be repaired automatically.

## Adversarial self-audit

- Confirmed both target repos were empty with `git ls-remote` before preparing.
- Confirmed the credential helper returns credentials without printing values.
- Confirmed `origin` in control-plane is still the old orchestration URL and no
  remote was removed or renamed.
- Confirmed the full preserved control-plane history passes the security driver,
  including the single accepted historical exception.
- Confirmed the wrapper failure reproduces with the exact arguments Git supplies
  to `pre-push`, so it is not a simulation artifact.
- Confirmed no bypass (`--no-verify`, hook-path override, SSH substitution or
  embedded credential) was used.

## Remaining risks and follow-up

1. The pre-push wrapper must be corrected before publication. A minimal fix is
   to stop forwarding Git hook arguments to the Python driver (the driver reads
   refs from stdin and needs no remote arguments), or to make the driver accept
   them. Either change needs an authorized control-plane commit and an authorized
   docs commit, since both wrappers carry the defect.
2. The docs repository already has its authorized extra commit; fixing its
   wrapper would require one more owner-authorized docs commit.
3. Write access remains unproven until a successful push.
4. Local hooks remain bypassable; remote-side enforcement is still an owner
   decision.
5. The historical report and its fingerprint exception remain unchanged.

## Owner decisions required

1. Authorize a follow-up change to the pre-push wrappers (control-plane and docs)
   and the corresponding local commits, then re-run publication.
2. Alternatively, explicitly authorize a different non-bypass remediation and
   state the commit budget for both repositories.
3. Confirm that write credentials should be used as-is once the gate passes.
4. Confirm whether remote-side branch protection/CI is desired after publication.

## Report saved

- Canonical absolute path: `/home/iffi/dev-workspace/docs/workspace/implementation/GITHUB-REPOSITORY-PUBLICATION.md`
- Category: `implementation`
- SHA-256: computed after write, not embedded.
- Validation: all content/security validators and the 16-case hook suite pass;
  publication is blocked solely by the wrapper argument defect described above.
