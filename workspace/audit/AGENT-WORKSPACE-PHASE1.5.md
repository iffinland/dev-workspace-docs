# Phase 1.5 — Full VPS documentation and path migration audit

Date: 2026-10-30
Status: PASS (all verified ACTIVE paths corrected; intentional references remain)
Executing agent: unknown (not established by evidence in this session; recorded as unknown rather than inferred from the CLI/profile)
Authorized actions: local documentation edits only. No commit, push, branch change, build, deploy, publication, runtime, node, wallet or secret access.

## 1. Scope

Audited and migrated active documentation and configuration so the OpenCode/DeepSeek
development environment reflects the actual VPS layout:

- `/home/iffi/dev-workspace/control-plane/`
- `/home/iffi/dev-workspace/projects/`
- project-level `AGENTS.md` files
- agent/role instructions, platform packs, skills and index
- workflow, handoff, registry, environment and template documents
- active scripts and OpenCode configuration
- existing backups and historical reports (read-only context)

Backups under `/home/iffi/dev-workspace/backups/agent-workspaces-20261030/`
were inspected for context only and were not modified.

## 2. Method

1. Enumerated and path-scanned the control plane and project trees.
2. Ran a case-insensitive old-path scan for `VsCodec`, `AI-Orchestration`,
   `00-SESSION-START`, `qortium-dev-workspace`, `projects/blogs.md` and
   `docs/blogs`.
3. Read every active file containing a candidate path.
4. Verified each proposed replacement against the real filesystem before editing.
5. Classified each finding, corrected verified ACTIVE paths, and left
   HISTORICAL/REFERENCE material unchanged.
6. Ran the existing validators plus an additional read-only audit for active
   absolute paths, relative Markdown links, registry rows and old-path residue.

## 3. Audit volume

| Metric | Count |
| --- | --- |
| Files enumerated/scanned in `control-plane/` + `projects/` (excludes `.git`, `node_modules`, `dist`, `build`) | 4,398 |
| Active Markdown files deep-validated | 81 |
| Control-plane active Markdown files (validator) | 37 |
| Control-plane required files (validator) | 20 |
| Active skills indexed | 15 |
| Raw old-path line occurrences found before correction | 2,555 |
| Rows in initial old-path hit set (distinct files) | 81 |
| Absolute paths verified in active docs | 32 |
| Relative Markdown links verified in active docs | 98 |
| OpenCode global config entries checked | 1 |
| Workflow shell script checked | 1 |

## 4. Old path categories found

1. Old Qortium workstation root: `/home/iffi/VsCodec-Projects/Qortium/...`.
2. Old Qortal workstation root: `/home/iffi/VsCodec-Projects/Qortal/...`.
3. Old orchestration root: `/home/iffi/VsCodec-Projects/AI-Orchestration/...`.
4. Obsolete session routers: `agents/00-SESSION-START.md`.
5. Obsolete project-context files: `projects/blogs.md`,
   `projects/nodefm-station.md`, `projects/qortium-united-community.md`.
6. Obsolete report roots: `.../docs/blogs/`, `.../docs/nodefm-station/`,
   `.../docs/qortium-united-community/`.
7. Old workspace shorthand in Qortal skills: `Qortal/qortal-dev-workspace`.
8. Preserved provenance paths inside historical reports, task artifacts, the
   Phase 1 migration record, and preserved reference repositories.

No Windows paths, old workstation IPs, obsolete tunnel credentials, obsolete
node endpoints, or obsolete OpenCode model/API configuration were found in active
guidance. `ENVIRONMENT.md` already contains the Phase 1 replacement policy.

## 5. Classification summary

| Classification | Files | Line-hits | Action |
| --- | --- | --- | --- |
| ACTIVE — corrected | 12 | 22 | Paths replaced with verified VPS targets |
| REFERENCE — intentional warning | 1 | 1 | Preserved; documents the legacy path as nonexistent |
| CURRENT-REFERENCE — valid current path/URL | 2 | 2 | Preserved; points at the preserved migration repo |
| HISTORICAL — provenance/evidence | 71 | 2,537 | Preserved unchanged |
| Total | 86 | 2,562 | — |

The ACTIVE count includes three references in `projects/Blogs/AGENTS.md` fixed
earlier in this session, plus four Qortal skill references found by a separate
`qortal-dev-workspace` scan. Those seven are additional to the original
2,555-hit set, giving 2,562 total references.

## 6. ACTIVE findings and corrections

### 6.1 Project entry points

| File | Original reference | Problem | Replacement | Verified |
| --- | --- | --- | --- | --- |
| `projects/Blogs/AGENTS.md` | `.../agents/00-SESSION-START.md`; `.../projects/blogs.md`; `.../docs/blogs/` | All targets absent | Control-plane AGENTS/WORKFLOW/GIT-AND-HANDOFF/REGISTRY/Qortium pack; project README + `docs/RELEASE.md`; `/home/iffi/dev-workspace/docs/` | All targets exist/readable |
| `projects/nodefm-station/AGENTS.md` | Same Qortium router/context/report roots | Targets absent | Same pattern; project README + `docs/`; workspace docs | All targets exist/readable |
| `projects/Qortium-United-Community/AGENTS.md` | Same Qortium router/context/report roots | Targets absent | Same pattern; project README + `docs/`; workspace docs | All targets exist/readable |
| `projects/shadow-archives-webportal-QORTAL/AGENTS.md` | `AI-Orchestration/AGENTS.md`; `Qortal/.../00-SESSION-START.md`; `projects/shadow-archives-webportal.md` | Targets absent | Control plane + Qortal pack; project README; preserved Qortal source reference; workspace docs | All targets exist/readable |

All four entry points now state that they are subordinate to the canonical control
plane and that no separate Qortium/Qortal session router exists in Phase 1.

### 6.2 Project README / roadmap references

| File | Line (pre-edit) | Original | Replacement | Verified |
| --- | --- | --- | --- | --- |
| `projects/Blogs/README.md` | 36–40 | Qortium context `.md` + `docs/blogs/` | README + `docs/RELEASE.md`; `/home/iffi/dev-workspace/docs/` | PASS |
| `projects/nodefm-station/README.md` | 64–67 | Qortium context `.md` + `docs/nodefm-station/` | README + `docs/`; `/home/iffi/dev-workspace/docs/` | PASS |
| `projects/Qortium-United-Community/README.md` | 105–108 | Qortium context `.md` + `docs/qortium-united-community/` | README + `docs/`; `/home/iffi/dev-workspace/docs/` | PASS |
| `projects/nodefm-station/docs/ROADMAP.md` | 309 | `.../docs/nodefm-station/` | `/home/iffi/dev-workspace/docs/` | PASS |

### 6.3 Active Qortal skill evidence references

| File | Line | Original | Replacement |
| --- | --- | --- | --- |
| `control-plane/skills/qortal/registered-name-owner-mode/SKILL.md` | 175 | `(workspace Qortal/qortal-dev-workspace)` | Preserved source `projects/qortal-dev-workspace` + branch `origin/agent/qwb-qortal-web-builders/phase-4-checkpoint-docs-20260916` |
| `control-plane/skills/qortal/inline-owner-editing/SKILL.md` | 177 | Same | Same |
| `control-plane/skills/qortal/qdn-content-crud/SKILL.md` | 230 | Same | Same |
| `control-plane/skills/qortal/private-chat-contact-form/SKILL.md` | 32–34 | `Qortal/qortal-dev-workspace/docs/shadow-archives-webportal/` | `projects/qortal-dev-workspace/...`; verify preserved evidence branch before citing |

Verification: the QWB evidence files exist on
`origin/agent/qwb-qortal-web-builders/phase-4-checkpoint-docs-20260916`; the
Shadow Archives Contact evidence exists on the preserved Shadow Archives
branches. The skills now describe that state instead of implying the evidence is
in the main checkout.

## 7. Exact files changed

| # | File | Change |
| --- | --- | --- |
| 1 | `projects/Blogs/AGENTS.md` | Rewritten routing (prior task in this session) |
| 2 | `projects/Blogs/README.md` | Context/report routing corrected |
| 3 | `projects/nodefm-station/AGENTS.md` | Rewritten routing and subordination |
| 4 | `projects/nodefm-station/README.md` | Context/report routing corrected |
| 5 | `projects/nodefm-station/docs/ROADMAP.md` | Report root corrected |
| 6 | `projects/Qortium-United-Community/AGENTS.md` | Rewritten routing and subordination |
| 7 | `projects/Qortium-United-Community/README.md` | Context/report routing corrected |
| 8 | `projects/shadow-archives-webportal-QORTAL/AGENTS.md` | Rewritten Qortal routing and subordination |
| 9 | `control-plane/skills/qortal/registered-name-owner-mode/SKILL.md` | Evidence path clarified |
| 10 | `control-plane/skills/qortal/inline-owner-editing/SKILL.md` | Evidence path clarified |
| 11 | `control-plane/skills/qortal/qdn-content-crud/SKILL.md` | Evidence path clarified |
| 12 | `control-plane/skills/qortal/private-chat-contact-form/SKILL.md` | Evidence path clarified |

No application source code was modified. `Qortal-Video-Bridge` and
`wordpress-to-qortal` were clean and have no project `AGENTS.md`; they inherit
the control plane. Preserved reference repositories
(`projects/qortal-dev-workspace`, `projects/ai-development-orchestration`) were
not modified.

## 8. Exact old-to-new path mappings

| Old target | New target |
| --- | --- |
| `/home/iffi/VsCodec-Projects/Qortium/qortium-dev-workspace/agents/00-SESSION-START.md` | `/home/iffi/dev-workspace/control-plane/AGENTS.md` + `WORKFLOW.md` + `GIT-AND-HANDOFF.md` + `PROJECT-REGISTRY.md` + `platforms/qortium/README.md` |
| `/home/iffi/VsCodec-Projects/Qortal/qortal-dev-workspace/agents/00-SESSION-START.md` | `/home/iffi/dev-workspace/control-plane/AGENTS.md` + `WORKFLOW.md` + `GIT-AND-HANDOFF.md` + `PROJECT-REGISTRY.md` + `platforms/qortal/README.md` |
| `/home/iffi/VsCodec-Projects/AI-Orchestration/AGENTS.md` | `/home/iffi/dev-workspace/control-plane/AGENTS.md` |
| `.../projects/blogs.md` | `/home/iffi/dev-workspace/projects/Blogs/README.md` + `.../docs/RELEASE.md` (canonical context pending Phase 2) |
| `.../projects/nodefm-station.md` | `/home/iffi/dev-workspace/projects/nodefm-station/README.md` + `.../docs/` |
| `.../projects/qortium-united-community.md` | `/home/iffi/dev-workspace/projects/Qortium-United-Community/README.md` + `.../docs/` |
| `.../projects/shadow-archives-webportal.md` | `/home/iffi/dev-workspace/projects/shadow-archives-webportal-QORTAL/README.md` |
| `/home/iffi/VsCodec-Projects/Qortium/docs/blogs/` | `/home/iffi/dev-workspace/docs/` |
| `/home/iffi/VsCodec-Projects/Qortium/docs/nodefm-station/` | `/home/iffi/dev-workspace/docs/` |
| `/home/iffi/VsCodec-Projects/Qortium/docs/qortium-united-community/` | `/home/iffi/dev-workspace/docs/` |
| `Qortal/qortal-dev-workspace` (skill shorthand) | `projects/qortal-dev-workspace` (preserved source; branch evidence) |

No speculative one-to-one replacement was made where no verified target existed.
Missing canonical project contexts are explicitly marked "pending Phase 2 import"
rather than recreated.

## 9. HISTORICAL / REFERENCE inventory (unchanged)

### 9.1 REFERENCE (intentional, 1 file)

| File | Line-hits | Reason |
| --- | --- | --- |
| `control-plane/platforms/qortium/README.md` | 1 | Explicit warning that the legacy workstation path is historical and must not be assumed to exist |

### 9.2 CURRENT-REFERENCE (valid current targets, 2 files)

| File | Line-hits | Reason |
| --- | --- | --- |
| `control-plane/README.md` | 1 | Current GitHub repository URL for the migration source |
| `control-plane/ENVIRONMENT.md` | 1 | Current existing path `/home/iffi/dev-workspace/projects/ai-development-orchestration` |

### 9.3 HISTORICAL (71 files, 2,537 line-hits)

These files are historical reports, task artifacts, the Phase 1 migration record
or preserved reference repositories. They intentionally retain original paths as
provenance and were not modified.

| File | Line-hits |
| --- | --- |
| `control-plane/docs/migration/2026-10-30-phase1-control-plane.md` | 4 |
| `control-plane/reports/2026-09-13-document-inventory.json` | 750 |
| `control-plane/reports/2026-09-13-duplicates-and-conflicts.md` | 298 |
| `control-plane/reports/2026-09-13-files-changed.txt` | 37 |
| `control-plane/reports/2026-09-13-git-baseline.json` | 42 |
| `control-plane/reports/2026-09-13-git-final.txt` | 49 |
| `control-plane/reports/2026-09-13-phase-1-audit.md` | 5 |
| `control-plane/reports/2026-09-13-phase-2-qortal-node-b-tunnel.md` | 1 |
| `control-plane/reports/2026-09-13-phase-2-reference-sync-and-tunnels.md` | 4 |
| `control-plane/reports/2026-09-13-phase-2-skills-redesign.md` | 1 |
| `control-plane/tasks/orchestration/qortal-skill-promotion-20260914/AUDIT.md` | 1 |
| `control-plane/tasks/orchestration/qortal-skill-promotion-20260914/IMPLEMENTATION.md` | 1 |
| `control-plane/tasks/orchestration/qortal-skill-promotion-20260914/STATUS.json` | 3 |
| `control-plane/tasks/orchestration/qortal-skill-promotion-20260914/TASK.md` | 5 |
| `control-plane/tasks/qwb-qortal-web-builders/qwb-architecture-audit-20260915/AUDIT.md` | 4 |
| `control-plane/tasks/qwb-qortal-web-builders/qwb-architecture-audit-20260915/TASK.md` | 6 |
| `control-plane/tasks/qwb-qortal-web-builders/qwb-phase-0-1-20260915/IMPLEMENTATION.md` | 5 |
| `control-plane/tasks/qwb-qortal-web-builders/qwb-phase-0-1-20260915/STATUS.json` | 7 |
| `control-plane/tasks/qwb-qortal-web-builders/qwb-phase-0-1-20260915/TASK.md` | 7 |
| `control-plane/tasks/qwb-qortal-web-builders/qwb-phase-4-checkpoint-20260916/IMPLEMENTATION.md` | 2 |
| `control-plane/tasks/qwb-qortal-web-builders/qwb-phase-4-checkpoint-20260916/STATUS.json` | 3 |
| `control-plane/tasks/qwb-qortal-web-builders/qwb-phase-4-checkpoint-20260916/TASK.md` | 7 |
| `control-plane/tasks/qwb-qortal-web-builders/qwb-rc-7fd03fc-staging-confirmation-20260916/IMPLEMENTATION.md` | 1 |
| `control-plane/tasks/qwb-qortal-web-builders/qwb-rc-7fd03fc-staging-confirmation-20260916/STATUS.json` | 3 |
| `control-plane/tasks/qwb-qortal-web-builders/qwb-rc-7fd03fc-staging-confirmation-20260916/TASK.md` | 5 |
| `control-plane/tasks/shadow-archives-webportal/contact-phase-closure-20260915/IMPLEMENTATION.md` | 1 |
| `control-plane/tasks/shadow-archives-webportal/owner-runtime-checkpoint-20260913/IMPLEMENTATION.md` | 1 |
| `control-plane/tasks/shadow-archives-webportal/owner-runtime-checkpoint-20260913/TASK.md` | 4 |
| `projects/ai-development-orchestration/ENVIRONMENT.md` | 3 |
| `projects/ai-development-orchestration/GIT-AND-HANDOFF.md` | 1 |
| `projects/ai-development-orchestration/PROJECT-REGISTRY.md` | 9 |
| `projects/ai-development-orchestration/reports/2026-09-13-document-inventory.json` | 750 |
| `projects/ai-development-orchestration/reports/2026-09-13-duplicates-and-conflicts.md` | 298 |
| `projects/ai-development-orchestration/reports/2026-09-13-files-changed.txt` | 37 |
| `projects/ai-development-orchestration/reports/2026-09-13-git-baseline.json` | 42 |
| `projects/ai-development-orchestration/reports/2026-09-13-git-final.txt` | 49 |
| `projects/ai-development-orchestration/reports/2026-09-13-phase-1-audit.md` | 5 |
| `projects/ai-development-orchestration/reports/2026-09-13-phase-2-qortal-node-b-tunnel.md` | 1 |
| `projects/ai-development-orchestration/reports/2026-09-13-phase-2-reference-sync-and-tunnels.md` | 4 |
| `projects/ai-development-orchestration/reports/2026-09-13-phase-2-skills-redesign.md` | 1 |
| `projects/ai-development-orchestration/templates/TASK.md` | 1 |
| `projects/qortal-dev-workspace/AGENTS.md` | 2 |
| `projects/qortal-dev-workspace/README.md` | 3 |
| `projects/qortal-dev-workspace/agents/00-SESSION-START.md` | 1 |
| `projects/qortal-dev-workspace/agents/01-TASK-CLASSIFICATION.md` | 1 |
| `projects/qortal-dev-workspace/agents/README.md` | 3 |
| `projects/qortal-dev-workspace/docs/ai-orchestration/audits/2026-09-13-universal-orchestration-audit.md` | 3 |
| `projects/qortal-dev-workspace/docs/architecture/qortal-dapp-development-standard.md` | 1 |
| `projects/qortal-dev-workspace/docs/shadow-archives-webportal/architecture/2026-09-11-phase-1b-implementation-plan.md` | 1 |
| `projects/qortal-dev-workspace/docs/shadow-archives-webportal/audits/2026-09-11-independent-workflow-v2-pre-commit-audit.md` | 2 |
| `projects/qortal-dev-workspace/docs/shadow-archives-webportal/bootstrap/2026-09-11-qortal-workflow-v2-bootstrap-report.md` | 7 |
| `projects/qortal-dev-workspace/docs/shadow-archives-webportal/handoffs/2026-09-11-shadow-archives-phase-1a-handoff.md` | 2 |
| `projects/qortal-dev-workspace/docs/shadow-archives-webportal/handoffs/2026-09-13-gallery-closure-checkpoint.md` | 6 |
| `projects/qortal-dev-workspace/docs/shadow-archives-webportal/handoffs/2026-09-13-gallery-host-bridge-media-rendering-owner-handoff.md` | 1 |
| `projects/qortal-dev-workspace/docs/shadow-archives-webportal/handoffs/2026-09-13-gallery-owner-workflow-index-coherence-owner-handoff.md` | 4 |
| `projects/qortal-dev-workspace/docs/shadow-archives-webportal/implementation/2026-09-11-owner-runtime-visual-correction-followup-report.md` | 1 |
| `projects/qortal-dev-workspace/docs/shadow-archives-webportal/implementation/2026-09-11-phase-1b-appshell-implementation-report.md` | 4 |
| `projects/qortal-dev-workspace/docs/shadow-archives-webportal/implementation/2026-09-11-phase-2a-qdn-read-pipeline-report.md` | 2 |
| `projects/qortal-dev-workspace/docs/shadow-archives-webportal/implementation/2026-09-11-phase-2b-owner-capability-host-validation-report.md` | 4 |
| `projects/qortal-dev-workspace/docs/shadow-archives-webportal/implementation/2026-09-12-phase-3a-gallery-publishing-report.md` | 2 |
| `projects/qortal-dev-workspace/docs/shadow-archives-webportal/release/2026-09-11-first-app-publication-readiness-report.md` | 5 |
| `projects/qortal-dev-workspace/docs/shadow-archives-webportal/remediation/2026-09-11-workflow-v2-audit-remediation-report.md` | 2 |
| `projects/qortal-dev-workspace/docs/shadow-archives-webportal/runtime/2026-09-13-gallery-host-read-contract-and-media-rendering-report.md` | 1 |
| `projects/qortal-dev-workspace/docs/shadow-archives-webportal/validation/2026-09-12-published-runtime-remediation-report.md` | 2 |
| `projects/qortal-dev-workspace/docs/shadow-archives-webportal/validation/2026-09-12-published-runtime-smoke-harness.cjs` | 2 |
| `projects/qortal-dev-workspace/docs/shadow-archives-webportal/validation/2026-09-12-real-qortal-host-validation-report.md` | 3 |
| `projects/qortal-dev-workspace/docs/workflows/deepseek-primary-work-model.md` | 1 |
| `projects/qortal-dev-workspace/docs/workflows/report-storage-policy.md` | 5 |
| `projects/qortal-dev-workspace/docs/workflows/workflow-v2.md` | 2 |
| `projects/qortal-dev-workspace/projects/shadow-archives-webportal.md` | 5 |
| `projects/qortal-dev-workspace/tools/validate-workspace.sh` | 1 |

## 10. Validation results

| Check | Result | Evidence |
| --- | --- | --- |
| Old-path search across active guidance (case-insensitive `vscodec`, `ai-orchestration`, `00-session-start`) | PASS with 2 intentional hits | Only the migration record (HISTORICAL) and Qortium platform warning (REFERENCE) |
| Every active absolute path exists | PASS | 32/32 paths |
| Every referenced active document readable | PASS | 0 unreadable files |
| Relative Markdown links resolve | PASS | 98/98 links across 81 active files |
| Control-plane validator | PASS | 37 active Markdown files, 74 relative links, 20 required files |
| Skills validator | PASS | 15 active skills indexed exactly once |
| Registry paths match directories | PASS | All 6 registered projects verified |
| Registry AGENTS presence matches declaration | PASS | Blogs, QUC, nodefm, shadow = Yes; QVB, wordpress = No |
| Platform routing | PASS | Qortal projects → `platforms/qortal`; Qortium projects → `platforms/qortium`; no cross-platform application |
| Project instructions subordinate to control plane | PASS | All four project entry points state subordination; no session-router claims |
| Scripts/configs free of obsolete paths | PASS | `scripts/env.sh` uses `$HOME/dev-workspace`; OpenCode config points at canonical AGENTS; no project script references old roots |
| `git diff --check` | PASS | No whitespace errors in changed repos |
| Build / commit / push / branch change | NOT PERFORMED (not authorized) | Working trees left uncommitted |

## 11. Unresolved ambiguities

1. **Canonical project contexts do not exist.** The registry marks them "Pending
   Phase 2 import". The migrated files point to each project's own README/docs
   and explicitly state the context is pending; nothing was invented.
2. **No canonical platform report root exists.** Per the owner reporting policy,
   `/home/iffi/dev-workspace/docs/` is used as the current report destination.
   This is an interim workspace convention, not a platform-defined root.
3. **`control-plane/platforms/qortium/README.md` retains one legacy absolute
   path.** It is an intentional warning that the path is historical and must not
   be assumed to exist. Classified REFERENCE, not routing.
4. **Historical provenance contains old paths by design.** The Phase 1 migration
   record, reports, task artifacts and preserved reference repositories were not
   rewritten.
5. **Some Qortal skill evidence lives only on preserved branches.** The skills
   now name the preserved source and branch instead of implying main-checkout
   availability; the evidence was verified on those branches.
6. **The running OpenCode session still holds the pre-edit project instruction
   text for any file loaded at session start.** On-disk content is corrected;
   a new session/restart is required for the updated project `AGENTS.md` to be
   injected.

## 12. Remaining migration risks

- Project documentation changes are local and uncommitted; they could be lost if
  the working trees are reset without owner action.
- Phase 2 import may introduce canonical project contexts and report roots;
  the interim README/workspace-docs routing must then be updated.
- Historical reports will continue to match old-path greps; future audits must
  keep excluding `reports/`, `tasks/`, the migration record and preserved
  reference repositories.
- The control-plane working tree already contained pre-existing uncommitted
  Phase 1 changes before this task. This task added changes only to the four
  Qortal skill files there; those pre-existing changes remain untouched.
- No runtime, QDN, node, wallet, secret, build, Git branch, commit or push action
  was performed or validated.

## 13. Report metadata

- Report path: `/home/iffi/dev-workspace/docs/workspace/audit/AGENT-WORKSPACE-PHASE1.5.md`
- Raw audit logs: `/tmp/opencode/phase15/` (ephemeral; not part of the durable
  workspace)
- Executing agent: unknown (not established by evidence; not inferred from the
  CLI or tool profile)
- Git state: uncommitted; no commit or push
