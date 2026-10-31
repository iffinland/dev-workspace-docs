# GitHub CI and branch protection audit — workspace/audit

Executing agent (registered role of the agent that actually produced this work): DeepSeek
Report/handoff writer (if different from the executing agent): DeepSeek (same)
Executing-agent evidence (how the real executor was established): Work was executed in this session by the model identified in-session as `deepseek/deepseek-flash`; the canonical control-plane contract, workflow, handoff rules, report-storage policy and `roles/DEEPSEEK.md` were read and followed. The CLI/orchestration profile is not used as identity.
Report type / category (audit, reports, research, implementation, validation, handoffs): audit
Project ID / central report root: cross-project / control plane → `/home/iffi/dev-workspace/docs/workspace/`
Exact application repository / branch / SHA: control-plane `main` @ `74cc44647b2e6796332059b33f6b6638000061bb`; central docs `main` @ `b188d0004d71018468b0d8e3aa1fd8529a524986`
Canonical report path `docs/<project>/<category>/<dated-name>.md` / SHA-256 / authorized remote evidence: `/home/iffi/dev-workspace/docs/workspace/audit/2026-10-30-GITHUB-CI-BRANCH-PROTECTION-AUDIT.md`; SHA-256 computed after write (not embedded); evidence is read-only GitHub API/local inspection only.

## Objective and exit criterion

Design a reliable, maintainable GitHub-side validation and branch protection
system for the private repositories `iffinland/dev-control-plane` and
`iffinland/dev-workspace-docs`, while verifying what the current GitHub account
and plan can actually enforce.

Result: **PARTIAL**. The audit and complete implementation plan are finished.
The blocking capability fact is that both repositories are private on a GitHub
Free personal account, where protected branches and rulesets are not available.
GitHub Actions itself is available for private repositories, but without a
plan upgrade (or a repository ownership/visibility change) CI can only be
advisory: required status checks and direct-push blocking cannot be enforced.

No GitHub setting, workflow, repository file, commit or push was changed by this
audit. Only this untracked report was created.

## Confirmed GitHub capabilities

Evidence used: authenticated GitHub REST API read-only queries, local Git
inspection, an Actions-capable local simulation, and the authoritative GitHub
documentation linked below. No credential value was printed or written.

### Repository and plan facts

| Fact | Confirmed result | Evidence |
| --- | --- | --- |
| Both repositories are private | Yes; `private: true`, `visibility: private` | REST repository metadata |
| Default branch | `main` on both | REST metadata and `git ls-remote --symref` |
| Current branch protection | `main` is **not protected** on either repository | `GET /repos/.../branches/main` returned `protected: false` |
| Repository rulesets | **Not available on the current plan** | `GET /repos/.../rulesets` returned HTTP 403: "Upgrade to GitHub Pro or make this repository public to enable this feature." The endpoint's accepted permission was `metadata=read`, so this is a plan gate, not a missing token scope. |
| Account plan | GitHub Free personal account (inferred from the rulesets plan gate and the plan documentation) | GitHub plan documentation: GitHub Pro is the first personal plan that adds protected branches for private repositories |
| Authenticated permission on both repositories | `admin`, `maintain`, `push`, `triage`, `pull` all true | REST repository metadata |
| Actions availability in principle | Available for private repositories on Free, consuming included compute | GitHub Actions billing documentation |
| Actions current enablement / defaults / runners / secrets / billing | **Unverified** | The current fine-grained token lacks Administration/Actions read; those REST endpoints returned 403. Owner UI or a temporary admin/actions-read token is required. |
| Existing workflow files | None in either checked-out repository | Local tree inspection; no `.github` directory |
| Existing CI protection | None confirmed | Branch unprotected; rulesets plan-gated; no workflow files |

### Current token limitation

The credential helper supplies a fine-grained personal access token. It is
sufficient for repository content read/write and allowed this audit to read
repository metadata and branch state. It is **not** sufficient to read
Administration or Actions settings:

```text
branch protection endpoint: HTTP 403, accepted permission administration=read
actions permissions endpoint: HTTP 403, accepted permission administration=read
rulesets endpoint: HTTP 403, plan upgrade message, accepted permission metadata=read
```

The audit therefore distinguishes confirmed branch/ruleset facts from the
unverified Actions settings. It does not treat a 403 as proof that a feature is
absent unless the response explicitly states a plan limitation.

### Authoritative plan and feature documentation

- GitHub plans (GitHub Pro adds protected branches for private personal
  repositories): <https://docs.github.com/en/get-started/learning-about-github/githubs-plans>
- About protected branches and required status checks:
  <https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-protected-branches/about-protected-branches>
- About rulesets:
  <https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets>
- GitHub Actions billing and private-repository included compute:
  <https://docs.github.com/en/billing/concepts/product-billing/github-actions>
- `GITHUB_TOKEN` is limited to the repository that contains the workflow:
  <https://docs.github.com/en/actions/concepts/security/github_token>
- Reusable workflows and cross-repository references:
  <https://docs.github.com/en/actions/how-tos/reuse-automations/reuse-workflows>

## CI architecture

### Why this is a cross-repository problem

The control-plane repository owns the shared security implementation:

```text
control-plane/tools/validate_security.py    scanner, exception engine, self-test
control-plane/tools/git_hooks.py            staged/outgoing Git hook driver
control-plane/tools/security-exceptions.json
control-plane/tools/test_git_hooks.py       hook and wrapper regression tests
control-plane/.githooks/pre-commit
control-plane/.githooks/pre-push
```

The docs repository contains only thin hook wrappers and its own empty exception
policy. Its wrappers locate the shared driver through a sibling checkout or
`CONTROL_PLANE_ROOT`. A docs-only CI checkout therefore cannot run the shared
validator without obtaining control-plane code.

`GITHUB_TOKEN` is scoped to the repository running the workflow, so the docs
workflow cannot clone the private control-plane repository with it. A read-only
cross-repository credential is required if the validator remains single-source.

### Recommended architecture

Use two independent workflows, one per repository, both producing a stable
job-level check name:

| Repository | Workflow/job | Required check name |
| --- | --- | --- |
| `dev-control-plane` | `.github/workflows/validate.yml` / `control-plane-validation` | `control-plane-validation` |
| `dev-workspace-docs` | `.github/workflows/validate.yml` / `docs-validation` | `docs-validation` |

Both workflows:

- run on `pull_request` targeting `main`, `push` to `main`, and
  `workflow_dispatch`;
- use `permissions: contents: read`;
- use a concurrency group with cancel-in-progress;
- set a short job timeout;
- use only the preinstalled runner Git/Python and pinned `actions/checkout`;
- do not upload artifacts, use caches, install packages or call external
  services;
- do not use `paths` filters, because a skipped required workflow leaves its
  check pending and blocks merges.

### Control-plane workflow plan

Proposed job steps:

1. Check out the repository with `fetch-depth: 0` and
   `persist-credentials: false`.
2. Assert the preinstalled Python is at least 3.10.
3. `python3 tools/validate_control_plane.py`
4. `python3 tools/validate_skills.py`
5. `python3 tools/validate_security.py --self-test`
6. `python3 tools/validate_security.py .`
7. `python3 tools/test_git_hooks.py`
8. Run the shared driver over the reachable history:
   `printf 'refs/heads/main %s refs/heads/main %s\n' "$GITHUB_SHA" "<zero-sha>" | python3 tools/git_hooks.py pre-push --repo .`

With no docs sibling checkout, the test suite runs 20 cases. This was simulated
locally in a disposable copy: all 20 passed, plus all four validators/scans.

### Docs workflow plan

The recommended single-source option checks out both repositories into sibling
paths:

```text
${{ github.workspace }}/docs
${{ github.workspace }}/control-plane
```

Proposed job steps:

1. Check out the docs repository into `docs` with `fetch-depth: 0` and
   `persist-credentials: false`.
2. Check out `iffinland/dev-control-plane` into `control-plane` using a
   read-only `CONTROL_PLANE_READ_TOKEN`, at a reviewed commit SHA, with
   `fetch-depth: 0` and `persist-credentials: false`.
3. `python3 control-plane/tools/validate_security.py --exceptions docs/.security-exceptions.json docs`
4. `python3 control-plane/tools/test_git_hooks.py`
5. Run the shared driver over docs history:
   `printf 'refs/heads/main %s refs/heads/main %s\n' "$GITHUB_SHA" "<zero-sha>" | python3 control-plane/tools/git_hooks.py pre-push --repo docs`

The sibling layout is required by `test_git_hooks.py`:
`CONTROL_PLANE.parent / "docs"`. With both checkouts present, all 24 tests run,
including both wrappers and real local Git pushes. This was simulated locally:
all 24 passed, and the docs scan passed over 25 files.

The docs workflow needs a fine-grained token with **Contents: Read** on
`dev-control-plane` only, stored as a docs repository secret. It should not
have write, Actions, Administration or other scopes. Set an expiry and rotate
it. It is used only by the control-plane checkout step; both checkouts should
disable credential persistence so later steps do not inherit it.

### Cross-repository dependency alternatives

| Option | Advantages | Costs / risks | Recommendation |
| --- | --- | --- | --- |
| Read-only token + control-plane checkout | Single source of truth; docs CI uses the exact shared validator and tests | One cross-repo secret; must rotate; PR workflow code with secret access is a trust boundary | **Recommended** |
| Vendor a pinned copy of the validator into docs | No cross-repo secret; simpler workflow | Duplicated security logic; drift risk; manual sync | Fallback if cross-repo secrets are unacceptable |
| Shared third `dev-security` repository | Clean ownership boundary | New repository, access management, pinning and maintenance; outside current two-repo scope | Future option only |
| Reusable workflow in control-plane | Reuses workflow configuration | Still needs cross-repo access; more moving parts; check naming/ref coupling | Not simpler than checkout |

Pinning the control-plane checkout to a reviewed commit SHA is safer than
following `main`; the tradeoff is a periodic pin update when security tooling
changes. If the owner prefers zero pin maintenance, use `main` and rely on
control-plane's own protected PR checks, accepting that a control-plane change
can affect docs CI immediately.

### Workflow skeleton

This is a plan, not a created file:

```yaml
name: validate
on:
  pull_request:
    branches: [main]
  push:
    branches: [main]
  workflow_dispatch:
permissions:
  contents: read
concurrency:
  group: validate-${{ github.repository }}-${{ github.ref }}
  cancel-in-progress: true
jobs:
  control-plane-validation:        # docs: docs-validation
    name: control-plane-validation # docs: docs-validation
    runs-on: ubuntu-latest
    timeout-minutes: 10
    steps:
      - uses: actions/checkout@<reviewed-full-sha>
        with:
          fetch-depth: 0
          persist-credentials: false
      - run: python3 -c 'import sys; assert sys.version_info >= (3, 10), sys.version'
      - run: python3 tools/validate_control_plane.py
      - run: python3 tools/validate_skills.py
      - run: python3 tools/validate_security.py --self-test
      - run: python3 tools/validate_security.py .
      - run: python3 tools/test_git_hooks.py
      - name: Scan reachable history
        run: |
          zero=0000000000000000000000000000000000000000
          printf 'refs/heads/main %s refs/heads/main %s\n' "$GITHUB_SHA" "$zero" \
            | python3 tools/git_hooks.py pre-push --repo .
```

The docs variant adds its own checkout at `path: docs`, checks out control-plane
at `path: control-plane` with the read-only token, omits the control-plane-only
validators, and runs the shared commands against `docs`.

## Branch protection architecture

### Current capability conclusion

On the current GitHub Free personal plan with private repositories:

- classic protected branches are not available;
- repository rulesets are not available;
- therefore required status checks cannot be made mandatory and direct pushes
  to `main` cannot be blocked by GitHub;
- workflows can still run and report failures, but a determined pusher can
  ignore them.

This is a plan capability limitation, not a workflow design problem.

### Recommended target after a plan upgrade

Use a repository ruleset per repository (preferred) or classic branch protection
if the owner prefers the simpler UI. Both require GitHub Pro for private
personal repositories. A ruleset has the advantage of being visible to
read-access users and layering cleanly, but classic protection is sufficient
for one `main` branch.

Target settings for `main`:

- **Require a pull request before merging** — this is the control that blocks
  direct pushes. Required approvals: `0` for a solo owner, or `1` if a second
  trusted reviewer exists. Do not configure a mandatory approval that cannot be
  satisfied.
- **Require status checks to pass** — require `control-plane-validation` in
  control-plane and `docs-validation` in docs.
- **Require branches to be up to date before merging** (strict) — ensures the
  checked result matches the merge base. This costs more runs but is appropriate
  for these small repositories.
- **Block force pushes**.
- **Restrict deletions**.
- **Do not allow bypassing / no bypass actors** — otherwise the administrator
  remains exempt and direct pushes remain possible.
- Optional: require linear history and conversation resolution. Do not require
  signed commits yet; existing history is unsigned and signing is not currently
  configured.

Rulesets on personal repositories do not provide the organization-only
"restrict who can push" actor list. The pull-request requirement with no bypass
is the correct substitute for a solo/private personal account. If role-based
push restrictions are required, the repositories must move to an organization
on GitHub Team or Enterprise; that changes ownership and URLs and is out of
scope without a separate owner decision.

### Required status checks and direct pushes

Requiring status checks alone does not reliably stop direct pushes. The
enforcement combination is:

```text
require pull request before merging
+ require the named status check
+ require branch up to date
+ no bypass actors / do not allow bypassing
+ block force pushes and deletions
```

After enabling, verify positively:

1. A direct push to `main` is rejected.
2. A PR with a failing job cannot be merged.
3. A PR with green checks can be merged normally.
4. The check names are stable and unique in each repository.
5. The `main` push workflow runs after merge as an audit.

### Plan comparison

| Option | Protection for private `main` | Cost / risk | Verdict |
| --- | --- | --- | --- |
| Stay on GitHub Free, private | No enforceable branch protection or rulesets; CI advisory only | No new cost; cannot satisfy the objective | Not sufficient |
| Upgrade personal account to GitHub Pro | Protected branches and rulesets available for private repos | Paid plan; no repository move or URL change | **Recommended** |
| Move repositories to a GitHub Team organization | Adds organization roles and push restrictions | Ownership/URL changes, org billing, migration work | Only if role-based controls are required |
| Make repositories public | Free protected branches for public repos | Exposes historical infrastructure detail and private reports; also unlocks public fork/PR surface | Not acceptable without sanitization/history decision |
| External CI without GitHub protection | Can run tests | Statuses do not block merges; no direct-push protection | Not sufficient alone |

## Existing history and security exception effects on CI

- Control-plane history currently constructs successfully through the full
  pre-push driver: 16 commits and 127 blobs, including the preserved historical
  report and its single exact-line exception.
- Docs history currently constructs successfully: 4 commits and 25 blobs; its
  exception policy is empty.
- The exception is keyed to repository-relative path, finding kind, line number
  and SHA-256 of the complete line. Reformatting that historical line, changing
  its line ending, or adding another address causes CI to fail closed.
- Use Ubuntu runners only. Do not run the validator under a checkout mode that
  rewrites line endings.
- CI should run the same driver and the same exception file as the local hooks,
  not an independent regex implementation.
- A full-history scan every run is affordable at the current size and catches
  deleted historical blobs. If history grows substantially, switch PR runs to
  an outgoing `base..head` scan and keep a manual/periodic full-history audit.
- GitHub secret scanning and push protection are not confirmed available for
  these private Free repositories; the local validator remains the primary
  security control.

## Runtime, cost and maintenance

Local measurements from disposable copies:

| Work | Measured local time |
| --- | --- |
| Control-plane validators and security scans | ~0.5 seconds |
| Control-plane-only hook tests (20 cases) | ~3.6 seconds |
| Both-repository hook tests (24 cases) | ~4.4 seconds |

A single Actions job should complete well under two minutes including checkout.
On GitHub Free for private repositories, included compute is $12 USD per month
(about 2,000 Linux minutes at the documented example rate); GitHub Pro raises
this to $18 USD. The two small jobs per relevant PR/merge are comfortably
within the included usage. No artifacts or caches are needed. Concurrency
cancellation avoids duplicate runs on rapid PR updates.

Maintenance items:

- pin `actions/checkout` to a reviewed full commit SHA;
- keep runner image and Python compatible (Ubuntu `ubuntu-latest`, Python >=3.10);
- rotate the docs cross-repository token before expiry;
- update the pinned control-plane SHA (if pinning) when security tooling changes;
- review workflow-file changes like any other security-sensitive code.

## Risks and limitations

1. **Current plan cannot enforce the target.** Without GitHub Pro (or an
   organization plan), CI failures do not block merges and direct pushes are
   possible. This is the primary unresolved gap.
2. **Admins can change the controls.** Even with protection, a repository admin
   can edit workflows, rulesets or settings. CI protects against unvalidated
   code changes, not against a hostile administrator. CODEOWNERS on
   `.github/workflows/**` and `tools/**` helps only if a second reviewer exists.
3. **Cross-repository secret exposure.** The docs workflow's read-only token is
   available to workflow code on same-repository PR branches. Use the minimum
   Contents: Read scope, no write permissions, a short expiry, and trusted
   collaborators only. Do not use `pull_request_target` to execute untrusted PR
   code with secrets.
4. **Actions enablement is unverified.** If Actions is disabled or restricted,
   required checks will remain pending and block every PR. Confirm Actions
   settings before enabling required checks.
5. **Default workflow token permissions are unverified.** Explicit
   `permissions: contents: read` in the workflow removes dependence on the
   repository default.
6. **Skipped workflows block merges.** Avoid `paths`/`branches` filters on the
   required workflow; a skipped check remains pending.
7. **Historical exception fragility.** Any edit to the accepted historical line
   fails CI by design and requires an owner-approved exception update.
8. **No local enforcement against a bypassing actor.** Local hooks remain
   bypassable with `--no-verify`; GitHub-side protection is the authoritative
   control once available.
9. **Cross-repository drift.** If the docs validator is vendored instead of
   shared, the copies can diverge. The recommended read-only checkout keeps one
   implementation.
10. **Current token cannot complete settings verification.** A temporary token
    with Administration and Actions read, or the owner UI, is needed to verify
    Actions enablement, default permissions, runners, secrets, deploy keys and
    billing state.

## Exact implementation phases

### Phase 0 — Owner decisions and baseline

1. Decide plan path: upgrade the personal account to GitHub Pro (recommended),
   move to an organization on Team/Enterprise, or accept advisory CI only.
2. Decide the solo-owner review policy: PR with 0 approvals, or require a second
   reviewer if one will be available.
3. Decide the docs cross-repository access method: read-only fine-grained token
   (recommended) or vendored validator fallback.
4. Obtain a temporary token or UI access with Administration and Actions read to
   verify Actions enablement, default workflow permissions, runners, secrets and
   billing. Record results.

### Phase 1 — Create and test control-plane workflow

1. Add `.github/workflows/validate.yml` with the control-plane job, explicit
   read-only permissions, pinned checkout, `fetch-depth: 0`, concurrency and
   timeout.
2. Open a PR changing only the workflow and a harmless documentation/test marker
   if needed. Confirm the job runs and reports `control-plane-validation`.
3. Confirm the workflow passes on a clean PR and fails on an intentionally
   secret-bearing or validator-breaking disposable PR branch (remove the test
   branch without merging it).

### Phase 2 — Create and test docs workflow

1. Create the read-only fine-grained token if chosen; store it only as
   `CONTROL_PLANE_READ_TOKEN` in the docs repository.
2. Add `.github/workflows/validate.yml` with the docs checkout at `path: docs`,
   the pinned control-plane checkout at `path: control-plane`, and the docs
   commands.
3. Open a docs PR and confirm `docs-validation` runs all 24 hook tests plus the
   docs scan.
4. Confirm the token is not persisted and is absent from later steps.

### Phase 3 — Upgrade and configure protection

1. After the plan upgrade is active, confirm rulesets/protected branches are
   available in both repositories.
2. Create the `main` ruleset or classic protection rule with the settings in the
   previous section.
3. Leave the bypass list empty and disable administrator bypass.
4. Require the exact check names that have already appeared in successful runs.

### Phase 4 — Positive and negative verification

1. Verify a direct push to `main` is rejected for the owner account.
2. Verify a PR with a deliberately failing check cannot be merged.
3. Verify a clean PR with passing checks can be merged normally.
4. Verify a force push and branch deletion are blocked.
5. Verify the `main` post-merge workflow still runs and passes.
6. Confirm local `core.hooksPath=.githooks`, the legacy
   `ai-development-orchestration` remote, and all existing branches remain
   unchanged.

### Phase 5 — Documentation and maintenance

1. Record the workflow files, protection settings, token scope/expiry and pinned
   action/control-plane SHAs in the control-plane governance documentation.
2. Add a manual `workflow_dispatch` full-history audit for both repositories, or
   a low-frequency scheduled audit if desired.
3. Review token expiry, action pins and Python/runner compatibility on a fixed
   schedule.
4. Record any future exception changes with the same owner-approved exact-line
   process.

## Owner decisions required

1. **Plan path:** upgrade to GitHub Pro (recommended), move to an organization
   Team/Enterprise, or accept advisory-only CI on Free.
2. **Review policy:** PR with 0 required approvals for solo work, or require one
   approval and designate a second reviewer.
3. **Cross-repository access:** approve a read-only fine-grained token in the
   docs repository, or choose the vendored-validator fallback.
4. **Control-plane checkout policy:** pin docs CI to a reviewed control-plane
   commit SHA, or follow `main` for lower maintenance with tighter coupling.
5. **Temporary settings access:** provide an Administration/Actions-read token
   or UI verification for Actions enablement, default workflow permissions,
   runners, secrets and billing.
6. **Optional hardening:** whether to require linear history, conversation
   resolution, signed commits later, or add CODEOWNERS for workflow and tooling
   paths.

## Files changed

- Created this untracked audit report:
  `/home/iffi/dev-workspace/docs/workspace/audit/2026-10-30-GITHUB-CI-BRANCH-PROTECTION-AUDIT.md`
- No workflow, repository, GitHub setting, commit, push, remote or runtime file
  was changed.
- The previously untracked
  `workspace/implementation/2026-10-30-GITHUB-PUBLICATION-COMPLETION.md` remains
  untracked and unchanged.

## Checks not executed and why

- No GitHub settings, rulesets, branch protection, Actions enablement, secrets,
  runners, visibility, branch protection or Actions settings were changed; not
  authorized.
- No workflow file was created or committed; not authorized.
- No push, pull, fetch, merge, tag, release or deployment was performed.
- No Qortal/Qortium runtime, node, QDN, wallet, signing or transaction action
  was performed.
- Actions run history, current Actions enablement/default permissions/runners
  and billing usage could not be read because the current fine-grained token
  lacks Administration/Actions read; this is recorded as an explicit limitation,
  not assumed.

## Adversarial self-audit

- Distinguished plan gating from token scope: the rulesets 403 explicitly names
  the GitHub Pro upgrade and the endpoint accepted only `metadata=read`, while
  the branch-protection/Actions 403s name missing token permissions.
- Cross-checked the plan conclusion against the authoritative GitHub plans page,
  which lists protected branches as a GitHub Pro feature for private
  repositories.
- Confirmed both branches are currently unprotected from the branch API rather
  than inferring from the ruleset error.
- Confirmed there are no existing workflow files locally and did not claim that
  Actions is enabled or disabled remotely.
- Simulated a control-plane-only checkout (20 tests) and a both-repository
  sibling checkout (24 tests); both passed, proving the proposed workflow
  commands are viable without changing canonical repositories.
- Verified the docs scan uses the docs exception file explicitly, because the
  standalone validator auto-loads exceptions only for paths under the
  control-plane root.
- Confirmed the proposed workflows use no third-party actions beyond a pinned
  official checkout, no package installation, no caches and no artifacts.
- Avoided path filters in the design because skipped required workflows leave
  checks pending and block merges.
- Redaction: no credential, token, raw infrastructure address or secret is
  present in this report or in the audit output.

## External actions (commit/push/tag/release/deploy/QDN write/transaction)

None. Only read-only GitHub API queries, local Git inspection, disposable
temporary-copy test simulations and this untracked report were performed.

## Report saved

- Canonical absolute path: `/home/iffi/dev-workspace/docs/workspace/audit/2026-10-30-GITHUB-CI-BRANCH-PROTECTION-AUDIT.md`
- Category: `audit`
- SHA-256: computed after write, not embedded.
- Validation: read-only API checks completed; disposable control-plane-only and
  both-repository test simulations passed; no canonical file, commit, push or
  GitHub setting changed.
