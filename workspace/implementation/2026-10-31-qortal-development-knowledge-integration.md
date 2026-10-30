# Qortal development knowledge integration — orchestration/qortal-development-knowledge-integration-20261031

Executing agent (registered role of the agent that actually produced this work): DeepSeek
Report/handoff writer (if different from the executing agent): DeepSeek
Executing-agent evidence (how the real executor was established): the runtime declares model ID `deepseek/deepseek-flash`; execution occurred in the OpenCode CLI harness. The harness is not the author.
Report type / category (audit, reports, research, implementation, validation, handoffs): implementation
Project ID / central report root: `orchestration` / `/home/iffi/dev-workspace/docs/workspace/`
Exact application repository / branch / SHA: `/home/iffi/dev-workspace/control-plane`; `agent/control-plane/qortal-development-knowledge-20261031`; base `main` @ `8c69e05f9e1d7f9bf7520b597f0fa7e58879df29`; implementation commit `dc3a96aee6d79866bd955849089e86c097d4287f`; evidence commit and branch head at report time `8c209c3dd873038c870e7cbb6cd488ac0d40a0e6`.
Canonical report path `docs/workspace/implementation/2026-10-31-qortal-development-knowledge-integration.md` / SHA-256 recorded after write / remote evidence: PR https://github.com/iffinland/dev-control-plane/pull/2

## Objective and exit criterion

Integrate the supplied Qortal development knowledge into the existing control
plane without replacing established workflows: classify the sources, add a
legacy dApp audit/repair capability and supporting references, integrate
generalized overnight practices, preserve Qortal/Qortium and security
boundaries, validate, commit/push a feature branch, open a PR, verify CI and
store this report centrally.

Exit criterion met: the new guidance is routed, provenance-labelled, validator-
and CI-clean, and the PR is open without merging to `main`.

## Inputs located and verified

| Source | Absolute path | SHA-256 | Handling |
| --- | --- | --- | --- |
| Overnight prompt | `/home/iffi/dev-workspace/docs/references/qortal-development/PROMPT-night-1.md` | `2dd0a817612f1954358ca8628953b1929ae8d70ab50da0570dea2912f2f08d60` | Read in full; project-specific product decisions rejected; generalized practice adopted. |
| Q-App guide | `/home/iffi/dev-workspace/docs/references/qortal-development/QORTAL_Q_APP_MASTER_GUIDE_FOR_LLMS.md` | `9a787b5070571c53ec732c2f94b1116c2d6dd50e8c365ab01ad1287ce2482868` | Read in full; version-sensitive claims classified, not adopted as platform truth. |

Both originals remain unmodified. A provenance record was added at
`/home/iffi/dev-workspace/docs/references/qortal-development/PROVENANCE.md`.

## Evidence by layer

| Layer | Evidence | Timestamp / result |
| --- | --- | --- |
| Source presence and hashing | `ls`, `wc`, `sha256sum` on both source files | 2026-10-30 UTC; both present, hashes above |
| Existing governance review | Read `AGENTS.md`, `WORKFLOW.md`, `GIT-AND-HANDOFF.md`, `PROJECT-REGISTRY.md`, `ENVIRONMENT.md`, `skills/README.md`, platform packs, report policy, templates and representative skills | Completed before editing |
| Conflict check | Compared third-party claims with `qdn-content-crud` and `qdn-derived-index-coherence`; read Qortium pack and skills | Existing verified skills win where they conflict; conflicts recorded |
| Structure validator | `python3 tools/validate_control_plane.py` | Passed: 43 active Markdown files, 120 relative links, 20 required files |
| Skills validator | `python3 tools/validate_skills.py` | Passed: 16 active skills indexed exactly once |
| Security scan | `python3 tools/validate_security.py .` | Passed: 92 files scanned |
| Git hook regression | `python3 tools/test_git_hooks.py` | Passed |
| Pre-commit / pre-push hooks | Repository hooks | Passed on both commits and the push |
| GitHub Actions (implementation) | `Validate control plane` run `41137018721` | completed / success for `dc3a96a` |
| GitHub Actions (final head) | `Validate control plane` run `41137188361` | completed / success for `8c209c3` |
| PR | GitHub API/PR page | https://github.com/iffinland/dev-control-plane/pull/2 (open, not merged) |

No live Qortal/Qortium node, host, application runtime or published QDN
resource was queried. This is a documentation/governance integration, not a
platform-behavior validation.

## Files created and modified

Created in `control-plane`:

- `skills/qortal/legacy-dapp-audit-repair/SKILL.md` — candidate audit/repair
  procedure: baseline, inventory, compatibility matrix, deterministic
  reproduction, prioritization, bounded repair, regression, release readiness
  and harvest.
- `platforms/qortal/references/README.md` — provenance, hashes, routing and
  reuse rules.
- `platforms/qortal/references/q-app-host-compatibility.md` — condensed host,
  bridge, QDN, security, performance and release checks; unverified/conflicting
  claims called out.
- `platforms/qortal/references/legacy-app-audit-checklist.md` — operational
  checklists and evidence fields.
- `platforms/qortal/references/source-adoption-analysis.md` — classification,
  adopted/rejected decisions and Qortal/Qortium applicability.
- `tasks/orchestration/qortal-development-knowledge-integration-20261031/TASK.md`
- `tasks/orchestration/qortal-development-knowledge-integration-20261031/STATUS.json`
- `tasks/orchestration/qortal-development-knowledge-integration-20261031/IMPLEMENTATION.md`

Modified in `control-plane`:

- `AGENTS.md` — routes legacy Qortal dApp audits to the new skill; states the
  condensed reference is candidate, not platform truth.
- `WORKFLOW.md` — adds a compact authorized unattended/overnight-session delta
  without duplicating or weakening existing sections.
- `platforms/qortal/README.md` — records the condensed reference, keeps full
  import deferred, adds routing.
- `platforms/README.md` — updates pack status without changing the import
  boundary.
- `skills/README.md` — indexes the new skill exactly once.
- `ENVIRONMENT.md` — registers the central source-reference path.
- `README.md` — adds the Qortal legacy audit reference to the main document list.

Created in the central docs repository:

- `docs/references/qortal-development/PROVENANCE.md`
- `docs/workspace/implementation/2026-10-31-qortal-development-knowledge-integration.md`
  (this report)

No application repository was modified. No Qortal Core, Hub, GO or VPS runtime
configuration was changed.

## Adopted and rejected recommendations

### Adopted

- Qortal host/QDN technical checks as **audit hypotheses to verify**: host
  detection, serving/base/routing, host messages and Back, theme/frame/mobile,
  search semantics and encoding, data availability/status, identifier/data
  contracts, publish/fee/batching/timeout behavior, money verification,
  encryption, storage isolation, sanitization, links/media and release
  verification.
- Legacy audit-and-repair methodology: inventory → compatibility verification →
  deterministic defect reproduction → prioritized repair → regression →
  release readiness.
- Data-contract preservation, additive versioning, tombstone compatibility and
  no automatic publish/pay.
- Generalized overnight practices: bounded outcome, no routine blocking,
  numbered follow-ups, serial money/data baseline, disjoint parallel work, one
  shared-runtime driver, budget stop condition and morning summary.

### Rejected or not adopted

- The third-party guide as authoritative or verified platform behavior.
- Q-Shop+/qplus-app product scope, design decisions, worktree path, branch,
  exact tooling commands, usage threshold, PR wording and project open
  questions.
- Copying the guide's templates/code into the control plane.
- Any universal app framework or giant abstraction.
- Transferring Qortal bridge/QDN/account rules to Qortium.
- Adding a background agent, scheduler or automatic task execution.

## Qortal versus Qortium applicability

- **Qortal-specific:** the audit checks for `qortalRequest`, q-apps.js,
  `APP`/`WEBSITE` serving, Qortal QDN services/identifiers, Qortal fees/names,
  encryption and release packaging.
- **Qortium:** no Qortal contract was added. Only platform-independent practices
  (audit phases, evidence fields, explicit write/money gates, artifact
  verification, responsive/accessibility checks, overnight discipline) are
  noted as candidates for separate Qortium validation against
  Qortium Home/Core source and runtime.
- **Platform-independent:** audit/reproduction/prioritization/regression,
  release-readiness evidence, capability harvest and authorized overnight
  session discipline.
- **Project-specific:** all Q-Shop+ product, design, branch and tooling
  decisions were retained in source provenance only.

## Unverified claims and compatibility risks

- The guide's version pins (Core 6.1.9, Hub 3.0.3, GO 3.0, Electron/WebView
  versions) were not revalidated.
- Exact CSP header, sandbox flags, 60 s dialog/late-Accept behavior, timeout
  tables, cold-read duration, service caps, fees, chat retention, `pollOptions`
  behavior and unreleased Hub `develop` actions remain version-sensitive.
- `exactmatchnames` guidance conflicts with the observed behavior recorded in
  the verified `qortal-qdn-content-crud` skill; the existing skill wins until
  re-checked.
- `PUBLISH_MULTIPLE_QDN_RESOURCES` result shape is recorded as both reject and
  resolve; the existing verified skill's observation is retained.
- `FILE`/`IMAGE`/`DOCUMENT` private-encryption enforcement is Core-version
  specific.
- The new audit skill remains `candidate` until a real legacy app audit is
  completed with source/runtime evidence.

## Validator and CI results

- `validate_control_plane.py`: pass.
- `validate_skills.py`: pass, 16 skills.
- `validate_security.py`: pass, 92 files.
- `test_git_hooks.py`: pass.
- Pre-commit and pre-push security hooks: pass.
- GitHub Actions `Validate control plane`: success on `dc3a96a` (run
  `41137018721`) and on final head `8c209c3` (run `41137188361`).

## External actions

- Pushed branch `agent/control-plane/qortal-development-knowledge-20261031` to
  `https://github.com/iffinland/dev-control-plane.git`.
- Opened PR #2 against `main`; it remains open and unmerged.
- Wrote the report and provenance files in the canonical docs repository.
- No application, platform, server, QDN publication, signing, transaction,
  deployment, release or merge action was performed.

## Remaining blockers

None for the documentation integration. The remaining open items are follow-up
validation, not blockers:

1. Run the new candidate skill on a real legacy Qortal dApp and record
   before/after evidence to raise its maturity.
2. Re-check the version-sensitive host claims against current Core/Hub/GO
   source/runtime before promoting any reference note.
3. Have the owner/controller review and merge PR #2 if accepted.

## Report saved

- Canonical absolute path:
  `/home/iffi/dev-workspace/docs/workspace/implementation/2026-10-31-qortal-development-knowledge-integration.md`
- Category: `implementation`
- SHA-256: recorded in the docs commit / repository metadata after write.
