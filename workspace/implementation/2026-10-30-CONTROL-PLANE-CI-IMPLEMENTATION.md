# Control-plane CI implementation — workspace/implementation

Executing agent (registered role of the agent that actually produced this work): DeepSeek
Report/handoff writer (if different from the executing agent): DeepSeek (same)
Executing-agent evidence (how the real executor was established): Work was executed in this session by the model identified in-session as `deepseek/deepseek-flash`; the canonical control-plane contract, workflow, handoff rules, report-storage policy and `roles/DEEPSEEK.md` were read and followed. The CLI/orchestration profile is not used as identity.
Report type / category (audit, reports, research, implementation, validation, handoffs): implementation
Project ID / central report root: cross-project / control plane → `/home/iffi/dev-workspace/docs/workspace/`
Exact application repository / branch / SHA: control-plane `agent/control-plane/simple-ci-20261030` @ `ebe2744a643b10116bc021ff2d8e39098a7516c9` (local only; push rejected)
Canonical report path `docs/<project>/<category>/<dated-name>.md` / SHA-256 / authorized remote evidence: `/home/iffi/dev-workspace/docs/workspace/implementation/2026-10-30-CONTROL-PLANE-CI-IMPLEMENTATION.md`; SHA-256 computed after write (not embedded); no remote evidence because GitHub rejected the workflow push.

## Objective and exit criterion

Implement a minimal GitHub Actions workflow for `iffinland/dev-control-plane`
using the existing Python validators and Git-hook tests, run it on pull requests
and pushes to `main`, use read-only workflow permissions, avoid external
packages/secrets/complex infrastructure, and preserve the local security hooks.
Push through a dedicated feature branch, open a pull request, and verify actual
GitHub CI execution.

Result: **PARTIAL**. The workflow is implemented, committed locally on the
dedicated branch, and passes all local equivalent checks. GitHub rejected the
branch push before upload because the current personal access token does not
have permission to create or update workflow files. No remote branch, pull
request or CI run exists.

## Branch and commit

| Item | Value |
| --- | --- |
| Base | `main` @ `74cc44647b2e6796332059b33f6b6638000061bb` |
| Feature branch | `agent/control-plane/simple-ci-20261030` |
| Workflow commit | `ebe2744a643b10116bc021ff2d8e39098a7516c9` |
| Commit subject | `ci(control-plane): add minimal validation workflow` |
| Remote branch | Not created (push rejected) |
| Pull request | Not created (no remote branch) |

The commit ran the active local pre-commit hook without bypass.

## Workflow implemented

New file: `.github/workflows/validate.yml`

```text
name: Validate control plane
on:
  pull_request: branches [main]
  push: branches [main]
permissions:
  contents: read
job: control-plane-validation
runner: ubuntu-latest
timeout: 10 minutes
```

Steps:

1. Check out with `fetch-depth: 0` and `persist-credentials: false` using
   `actions/checkout@v4`.
2. Assert preinstalled Python is at least 3.10.
3. `python3 tools/validate_control_plane.py`
4. `python3 tools/validate_skills.py`
5. `python3 tools/validate_security.py --self-test`
6. `python3 tools/validate_security.py .`
7. `python3 tools/test_git_hooks.py`
8. Full reachable-history scan through `tools/git_hooks.py pre-push --repo .`
   with the current commit and a zero remote base.

The workflow uses no packages, pip installs, caches, artifacts, secrets,
third-party actions beyond the official checkout action, or external services.
The local `.githooks` files and `core.hooksPath=.githooks` were not changed.

## Local validation of the implemented workflow and branch

| Check | Result |
| --- | --- |
| YAML parse and required keys/commands | PASS: valid, 8 steps |
| Workflow permissions | PASS: `contents: read` |
| `validate_control_plane.py` | PASS: 38 Markdown files, 85 links, 20 required files |
| `validate_skills.py` | PASS: 15 skills indexed exactly once |
| `validate_security.py --self-test` | PASS |
| `validate_security.py .` | PASS: 84 files scanned, including the new workflow |
| `test_git_hooks.py` | PASS: 24/24 in the canonical workspace (docs sibling present) |
| Full-history pre-push scan of branch HEAD | PASS: 17 commits, 128 blobs |
| Pre-push hook during the attempted branch push | PASS: 17 outgoing commits, 128 blobs |
| `git diff --check` | PASS (no output) |
| `git fsck --full` | PASS (exit 0) |
| Local working tree | Clean on the feature branch |

The workflow was not executed by GitHub because the branch never reached the
remote.

## Push attempt and blocker

Attempted normal non-force push:

```text
git push -u dev-control-plane agent/control-plane/simple-ci-20261030
```

The local security pre-push hook passed, then GitHub rejected the transfer:

```text
! [remote rejected] agent/control-plane/simple-ci-20261030 -> agent/control-plane/simple-ci-20261030
(refusing to allow a Personal Access Token to create or update workflow
`.github/workflows/validate.yml` without `workflow` scope)
```

No bypass was attempted. No `--no-verify`, force push, alternate credential,
API workaround, settings change or other control bypass was used.

Verified remote state after the rejection:

- `refs/heads/agent/control-plane/simple-ci-20261030` does not exist.
- `dev-control-plane/main` remains at `74cc44647b2e6796332059b33f6b6638000061bb`.
- The legacy `origin` remote remains `https://github.com/iffinland/ai-development-orchestration.git`
  and was not touched.

## Smallest required owner action

Grant workflow-file write permission to the credential used for this repository,
then rerun the same push and PR steps:

- For the current fine-grained token: add repository permission
  **Workflows: Read and write** for `iffinland/dev-control-plane`.
- Alternatively, use a classic token with the `workflow` scope for this push,
  then return to the least-privilege token.

No workflow-code change is required. The exact next commands are:

```text
git push -u dev-control-plane agent/control-plane/simple-ci-20261030
```

Then create a pull request from `agent/control-plane/simple-ci-20261030` to
`main`. After the push succeeds, GitHub Actions should start the
`control-plane-validation` job for the pull request; the owner or an
Actions-readable credential can confirm the run.

## Actual GitHub CI result

Not executed. The workflow file was rejected before upload, so GitHub Actions
never received the branch or workflow. There is no run ID, check run or PR
status to report.

## Files changed

Local feature branch commit `ebe2744`:

- `.github/workflows/validate.yml` (new)

This report is new and untracked in the central docs repository. No other
repository file, GitHub setting, branch, remote or runtime was changed.

## Checks not executed and why

- GitHub Actions execution and PR creation were not possible because the
  workflow push was rejected for missing workflow permission.
- No merge into `main`, branch protection change, GitHub settings change or
  Actions configuration change was attempted; out of scope or blocked.
- No legacy repository change, force operation, history rewrite or Qortium
  runtime action was performed.
- No alternate credential or bypass was used.

## Adversarial self-audit

- Confirmed the local branch contains exactly one new commit on top of the
  published `main` SHA.
- Confirmed the workflow uses only read permissions and no secrets or external
  packages.
- Confirmed the local security hooks still pass and the pre-push hook ran during
  the failed push.
- Confirmed the failure is GitHub's workflow-scope guard, not a network or
  authentication failure and not a workflow syntax error.
- Confirmed no remote feature branch was created and remote `main` is unchanged.
- Confirmed no force option, `--no-verify`, credential substitution, API
  workaround or settings change was used.
- Confirmed the legacy orchestration remote was not modified.

## External actions (commit/push/tag/release/deploy/QDN write/transaction)

- One local commit on the dedicated feature branch.
- One attempted non-force branch push to `dev-control-plane`; rejected before
  remote branch creation.
- No successful push, no PR, no merge, no tag, release, deployment, QDN write,
  signing, transaction or server change.
- No GitHub settings, visibility, branch protection or Actions settings change.

## Remaining risks and follow-up

1. The workflow file cannot be published until the credential has Workflows
   write permission (or the owner pushes it manually).
2. GitHub Free private repositories do not provide enforceable branch protection
   or required status checks. This CI is advisory and complements, but does not
   replace, the local hooks.
3. Once pushed, the first PR run must be checked for the actual GitHub Actions
   result and the `control-plane-validation` check name.
4. The docs wrapper tests are not part of this control-plane-only workflow;
   running them would require a cross-repository checkout/token and was
   deliberately excluded for simplicity.

## Report saved

- Canonical absolute path: `/home/iffi/dev-workspace/docs/workspace/implementation/2026-10-30-CONTROL-PLANE-CI-IMPLEMENTATION.md`
- Category: `implementation`
- SHA-256: computed after write, not embedded.
- Validation: local YAML, validators, security scan, hook tests, full-history
  scan and fsck pass; remote publication and CI verification are blocked solely
  by the token's missing workflow permission.
