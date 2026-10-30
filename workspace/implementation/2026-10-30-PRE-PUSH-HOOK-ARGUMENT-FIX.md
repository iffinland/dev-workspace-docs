# Pre-push hook argument fix — workspace/control-plane and docs

Executing agent (registered role of the agent that actually produced this work): DeepSeek
Report/handoff writer (if different from the executing agent): DeepSeek (same)
Executing-agent evidence (how the real executor was established): Work was executed in this session by the model identified in-session as `deepseek/deepseek-flash`; the control-plane contract, workflow, handoff rules, report-storage policy and `roles/DEEPSEEK.md` were read and followed. The CLI/orchestration profile is not used as identity.
Report type / category (audit, reports, research, implementation, validation, handoffs): implementation
Project ID / central report root: cross-project / control plane → `/home/iffi/dev-workspace/docs/workspace/`
Exact application repository / branch / SHA: control-plane `main` @ `74cc44647b2e6796332059b33f6b6638000061bb`; central docs `main` @ `9bee5ffaeab9ef344941eb3c26a4f76ca60ba872` (neither pushed)
Canonical report path `docs/<project>/<category>/<dated-name>.md` / SHA-256 / authorized remote evidence: `/home/iffi/dev-workspace/docs/workspace/implementation/2026-10-30-PRE-PUSH-HOOK-ARGUMENT-FIX.md`; SHA-256 computed after write (not embedded); no remote evidence (push not authorized)

## Objective and exit criterion

Fix the confirmed pre-push wrapper argument defect in both the `control-plane`
and central `docs` repositories so a real `git push` can invoke the shared
security driver, without bypassing the hook. Add regression tests that exercise
Git's actual pre-push invocation format, run the relevant validators and
security checks, and make exactly one corrective local commit per repository.

Exit criterion met: both wrappers accept Git's two positional pre-push arguments
(`<remote-name> <remote-url>`), actual local `git push` invocations pass the
hook for clean history and block secret-bearing history, and all targeted
validation passes.

## Root cause

Git invokes `pre-push` with two positional arguments (remote name and remote
URL) and supplies ref data on stdin. Both wrappers forwarded those positional
arguments to `tools/git_hooks.py`, whose argument parser accepts only the hook
name plus `--repo`/`--exceptions`. A real push therefore exited 2 before
transfer with `unrecognized arguments`. The shared driver reads ref data from
stdin and does not need the remote name or URL.

The fix removes the unnecessary `"$@"` forwarding from both `pre-push`
wrappers. The driver and the security policy were not weakened. `pre-commit`
was unchanged because Git supplies no positional arguments to that hook.

## Exact commits

| Repository | Branch | Parent SHA | Corrective commit SHA | Change |
| --- | --- | --- | --- | --- |
| control-plane | `main` | `5b9f9e3813aac15a739ce85e027c46c8b572a2c6` | `74cc44647b2e6796332059b33f6b6638000061bb` | Wrapper fix plus regression coverage |
| docs | `main` | `6589f2e625831b9b50c36aa7ee83f56f04437898` | `9bee5ffaeab9ef344941eb3c26a4f76ca60ba872` | Wrapper fix |

Preserved unchanged: `phase1/control-plane` and
`agent/shadow-archives/qdn-grouped-publish-outcome-20261001` in control-plane,
the full docs history, all historical reports, and
`control-plane/tools/security-exceptions.json` (the single accepted historical
exception remains byte-for-byte unchanged).

## Evidence by layer

### 1. Defect reproduction before the fix

A disposable repository made one clean commit, added a local bare remote, set
`core.hooksPath` to the canonical control-plane hooks, and ran a real
`git push origin main`. The wrapper forwarded `origin <local-remote-path>` to
`git_hooks.py`, which exited 2 with `unrecognized arguments`. This confirmed the
defect using the exact argument shape Git supplies, not a simulation artifact.

### 2. Source changes

- `control-plane/.githooks/pre-push`: stopped forwarding `"$@"` to the driver.
- `docs/.githooks/pre-push`: same minimal change.
- `control-plane/tools/test_git_hooks.py`: added regression coverage for both
  wrappers.

No validator, exception, policy, platform or runtime file was modified.

### 3. Regression tests

`python3 tools/test_git_hooks.py` passed **24/24** cases. The original 16 cases
remain. Eight new cases cover both repositories:

- wrapper invoked with Git's actual positional arguments and stdin ref data:
  clean history passes;
- same actual invocation with a secret-bearing outgoing commit: blocked;
- a real `git push origin main` into a disposable local bare remote through
  each repository's actual hooks directory: clean push passes;
- same real push with a secret-bearing commit: blocked.

All disposable repositories and remotes were created under the test's temporary
directory. The canonical repositories were not pushed to any remote.

### 4. Validators and security checks

| Check | Result |
| --- | --- |
| `python3 tools/test_git_hooks.py` | PASS: 24/24 |
| `python3 tools/validate_control_plane.py` | PASS: 38 Markdown files, 85 links, 20 required files |
| `python3 tools/validate_skills.py` | PASS: 15 skills indexed exactly once |
| `python3 tools/validate_security.py --self-test` | PASS |
| `python3 tools/validate_security.py .` (control-plane) | PASS: 83 files |
| `python3 tools/validate_security.py .` (docs) | PASS: 23 files before this report |
| Full-history pre-push scan, control-plane `main` | PASS: 16 commits, 127 blobs |
| Full-history pre-push scan, docs `main` | PASS: 3 commits, 23 blobs |
| `git diff --check` in both repositories | PASS (no output) |
| Pre-commit security hook during both corrective commits | PASS |

The full-history scans include the preserved historical report and its single
approved exception. No raw infrastructure value or credential is reproduced in
this report or terminal evidence.

### 5. Post-fix state

- Control-plane `main` is clean at `74cc446...`; `phase1/control-plane` remains
  at its prior SHA.
- Docs `main` is at `9bee5ff...`; only the pre-existing
  `workspace/implementation/GITHUB-REPOSITORY-PUBLICATION.md` and this new
  report are untracked.
- Both repositories still use `core.hooksPath=.githooks`.
- No remote, branch, tag or GitHub state was changed.

## Files changed

Control-plane corrective commit `74cc446`:

- `.githooks/pre-push`
- `tools/test_git_hooks.py`

Docs corrective commit `9bee5ff`:

- `.githooks/pre-push`

This report is intentionally not part of a separate commit, per the task's
"do not create an additional commit solely for the final report" constraint.

## Checks not executed and why

- No GitHub push, dry-run remote write, or write-access test was performed;
  remote changes were not authorized.
- No force operation, history rewrite, tag or release was performed.
- No Qortal/Qortium source, runtime, node, QDN, signing or transaction action
  was performed; out of scope.
- No branch protection, GitHub Actions, repository visibility or remote hook
  configuration was changed; out of scope.
- No full application build or platform runtime matrix was run; this bounded
  fix changes only local Git hook argument handling and its tests.

## Adversarial self-audit

- Reproduced the original failure through a real `git push`, not only a direct
  driver call.
- Confirmed the wrapper still blocks a secret under the corrected argument
  path; the fix does not convert a failing gate into a skip.
- Confirmed both repositories' wrappers, not just the control-plane wrapper,
  pass and block under actual Git invocation.
- Confirmed `tools/security-exceptions.json` is unchanged and the full history,
  including the excepted historical line, still passes.
- Confirmed no `--no-verify`, hook-path bypass, alternate validator, embedded
  credential or remote substitution was used.
- Confirmed commits were created through the installed pre-commit hook and both
  passed.
- Confirmed no unauthorized external action occurred.

## External actions (commit/push/tag/release/deploy/QDN write/transaction)

- Local commits only: one in control-plane (`74cc446`) and one in docs
  (`9bee5ff`).
- No push, force push, history rewrite, tag, release, deploy, QDN write,
  signature, transaction, issue mutation or server change.

## Remaining risks and follow-up

1. GitHub publication remains a separate owner-authorized step. The pre-push
   gate now accepts a real push in disposable local tests, but actual remote
   write access remains unproven until an authorized push is performed.
2. Remote-side enforcement or branch protection remains an owner decision; local
   hooks are still bypassable by a caller with filesystem access.
3. The prior publication report remains untracked and unchanged; this report
   does not supersede its historical facts, only the blocking hook defect.

## Report saved

- Canonical absolute path: `/home/iffi/dev-workspace/docs/workspace/implementation/2026-10-30-PRE-PUSH-HOOK-ARGUMENT-FIX.md`
- Category: `implementation`
- SHA-256: computed after write, not embedded.
