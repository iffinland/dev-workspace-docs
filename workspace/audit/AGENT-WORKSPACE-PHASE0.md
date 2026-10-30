# Agent Workspace Consolidation — Phase 0

Date: 2026-10-30
Phase: 0 — preservation and investigation only
Status: complete for Phase 0; no consolidation implementation started

Repositories:

- `/home/iffi/dev-workspace/projects/qortal-dev-workspace`
- `/home/iffi/dev-workspace/projects/ai-development-orchestration`

Backup root:

- `/home/iffi/dev-workspace/backups/agent-workspaces-20261030/`

---

## 1. Executive summary

Both working trees were clean before and after this phase. Their branches and
HEADs were not changed. Complete copies including `.git` were created and
verified. Remote metadata was fetched without pull, merge, rebase, reset,
checkout, or switch.

Two clear outcomes emerged:

1. **`ai-development-orchestration` has one safe migration base.**
   `origin/agent/shadow-archives/qdn-grouped-publish-outcome-20261001` at
   `aef3e1ed45c66ef2382e5cc794665d1720f7ad77` is a true superset of every other
   remote branch. It contains all newer skills, the skill validator, task
   templates/artifacts, governance updates, and the QWB/Qortium additions.

2. **`qortal-dev-workspace` has no single newer branch that contains all
   unmerged work.**
   Its newer branches split into Shadow Archives evidence lines and QWB
   documentation/evidence lines. The safe repository-wide migration base is the
   current `main` at
   `b8fd2f6fbfcff88c0ab9bc8fd116d5548679d2d2`, with all newer branch tips
   preserved and integrated in a later phase.

The Qortal platform material itself — `agents/`, `docs/architecture/`,
`docs/workflows/`, `templates/`, and `tools/` — is unchanged on every newer
Qortal branch. Newer Qortal commits add project contexts and historical
documentation/evidence, not new platform guidance.

---

## 2. Working-tree baseline

| Repository | Current branch | HEAD | Working tree | Remote |
| --- | --- | --- | --- | --- |
| `qortal-dev-workspace` | `main` | `b8fd2f6fbfcff88c0ab9bc8fd116d5548679d2d2` | clean | `https://github.com/iffinland/qortal-dev-workspace.git` |
| `ai-development-orchestration` | `agent/orchestration/bootstrap` | `c52e04841adfde062e579463a39bf0463609c8d5` | clean | `https://github.com/iffinland/ai-development-orchestration.git` |

Additional baseline facts:

- No stashes in either repository.
- No submodules in either repository.
- One working tree per repository.
- No local branches other than the checked-out branch.
- No tags in either repository.
- HEAD, branch, and working-tree state were verified again after the fetch and
  remained unchanged.

---

## 3. Backups

### 3.1 Location

```text
/home/iffi/dev-workspace/backups/agent-workspaces-20261030/
├── qortal-dev-workspace/
│   └── .git/
└── ai-development-orchestration/
    └── .git/
```

The backup was created with a complete recursive copy, including `.git` and all
working-tree files. Nothing was excluded.

### 3.2 Verification

| Check | qortal-dev-workspace | ai-development-orchestration |
| --- | --- | --- |
| `.git` present | Yes | Yes |
| Source/backup HEAD match | `b8fd2f6...` / `b8fd2f6...` | `c52e048...` / `c52e048...` |
| Source/backup branch match | `main` / `main` | `agent/orchestration/bootstrap` / same |
| Source/backup HEAD tree match | Yes | Yes |
| Tracked file count | 55 / 55 | 31 / 31 |
| Git refs in backup | 8 | 10 |
| `git fsck --full` | PASS | PASS |
| Backup working tree status | clean | clean |
| Full tree comparison | Identical except `.git/index` stat cache | Identical except `.git/index` stat cache |
| Backup size | 13 MB (`.git` 12 MB) | 904 KB (`.git` 420 KB) |

The `.git/index` stat cache differs because read-only `git status`/`fsck`
verification refreshed file stat information. Tracked content, commit objects,
trees, refs, and working-tree files match. This is not a content or history
difference.

### 3.3 Backup conclusion

Both backups are complete, readable, Git-valid, and contain the Git metadata and
tracked files. They are suitable as restoration points for Phase 0.

---

## 4. Remote fetch result

`git fetch origin --tags` was run in both repositories with terminal prompting
disabled. No pull, merge, rebase, reset, checkout, or switch was used.

Result:

- No new commits or branches were downloaded; both remotes were already current
  in the local remote-tracking refs.
- No tags were fetched because neither remote advertises tags.
- The live remote branch list was compared with cached remote-tracking refs:
  - `qortal-dev-workspace`: 6 live branches, exact match.
  - `ai-development-orchestration`: 8 live branches, exact match.
- Current branches and HEADs were unchanged.
- No tracked files were modified.

Remote default refs:

- `qortal-dev-workspace`: `origin/HEAD -> origin/main`
- `ai-development-orchestration`: `origin/HEAD -> origin/agent/orchestration/bootstrap`

---

## 5. Branch inventory

### 5.1 qortal-dev-workspace

Local branch:

| Ref | HEAD | Latest commit date |
| --- | --- | --- |
| `main` | `b8fd2f6fbfcff88c0ab9bc8fd116d5548679d2d2` | 2026-09-13T17:13:13+03:00 |

Remote branches:

| Ref | HEAD | Latest commit date | Ahead/behind `main` | Ancestor of `main`? |
| --- | --- | --- | --- | --- |
| `origin/main` | `b8fd2f6fbfcff88c0ab9bc8fd116d5548679d2d2` | 2026-09-13T17:13:13+03:00 | 0 / 0 | Yes |
| `origin/agent/shadow-archives-webportal/owner-runtime-docs-20260913` | `72b3c4d0e8c8018bde219b4940a2da6202c6b986` | 2026-09-15T13:42:51+03:00 | 2 / 0 | No |
| `origin/agent/shadow-archives-webportal/homepage-test-maintenance-docs-20260915` | `6f5cf53a55b454dba67b0f4fe78d7e44e75b1187` | 2026-09-15T15:30:25+03:00 | 4 / 0 | No |
| `origin/agent/shadow-archives/blog-publish-runtime-handoff-20261001` | `44cf3a5277f2de431b4d09ceae624c9ea8d3791b` | 2026-10-01T20:48:36+03:00 | 5 / 0 | No |
| `origin/agent/qwb-qortal-web-builders/phase-4-checkpoint-docs-20260916` | `694dff049707820ab3a8440692dabfc789f251ab` | 2026-10-01T20:49:35+03:00 | 3 / 0 | No |
| `origin/agent/qwb-qortal-web-builders/documentation-backfill-20261001` | `18d04c4102a276b67cec02aaf1450c8134d64f54` | 2026-10-01T20:48:49+03:00 | 1 / 0 | No |

Branch shape:

```text
b8fd2f6 main
├── 72b3c4d owner-runtime docs
│   ├── 6f5cf53 homepage maintenance docs
│   └── 44cf3a5 blog publish/runtime handoff
├── 694dff0 QWB phase-4 checkpoint docs (3 commits)
└── 18d04c4 QWB documentation backfill (1 commit, separate line)
```

`72b3c4d` is an ancestor of both `6f5cf53` and `44cf3a5`. The two QWB branches
are independent variants, not a parent/child chain.

### 5.2 ai-development-orchestration

Local branch:

| Ref | HEAD | Latest commit date |
| --- | --- | --- |
| `agent/orchestration/bootstrap` | `c52e04841adfde062e579463a39bf0463609c8d5` | 2026-09-13T17:12:35+03:00 |

Remote branches:

| Ref | HEAD | Latest commit date | Ahead/behind checked-out branch | Ancestor of latest? |
| --- | --- | --- | --- | --- |
| `origin/agent/orchestration/bootstrap` | `c52e04841adfde062e579463a39bf0463609c8d5` | 2026-09-13T17:12:35+03:00 | 0 / 0 | Yes |
| `origin/agent/shadow-archives-webportal/owner-runtime-handoff-20260913` | `c2cffde7a48b2495a896297c0634f4303a12bc12` | 2026-09-14T04:22:09+03:00 | 1 / 0 | Yes |
| `origin/agent/orchestration/qortal-skill-promotion-20260914` | `ba542bd3d263c84c91a1b00b684d2b2c0e7020e7` | 2026-09-14T04:34:54+03:00 | 2 / 0 | Yes |
| `origin/agent/orchestration/skills-quality-gate-20260914` | `359fcc5d92941c44a1029ca7018b88139cc76629` | 2026-09-14T12:58:04+03:00 | 3 / 0 | Yes |
| `origin/agent/orchestration/governance-checkpoint-20260914` | `9a611a21af469478a253fc664ab295d8c37f69d4` | 2026-09-15T13:42:51+03:00 | 5 / 0 | Yes |
| `origin/agent/orchestration/qwb-phase-4-checkpoint-20260916` | `1320e1648d4c18a50630b9db746b69f83c8b0e4d` | 2026-09-16T08:09:32+03:00 | 6 / 0 | Yes |
| `origin/agent/orchestration/qwb-rc-7fd03fc-staging-confirmation-20260916` | `4a5ac99690fafa6627eba9365ee5ec7a3047bec4` | 2026-09-16T08:35:42+03:00 | 7 / 0 | Yes |
| `origin/agent/shadow-archives/qdn-grouped-publish-outcome-20261001` | `aef3e1ed45c66ef2382e5cc794665d1720f7ad77` | 2026-10-01T20:49:08+03:00 | 9 / 0 | Yes |

Branch shape:

```text
c52e048 bootstrap
  -> c2cffde owner-runtime handoff
  -> ba542bd Qortal skill promotion
  -> 359fcc5 skills quality gate
  -> 9a611a2 governance checkpoint
  -> 1320e16 QWB phase-4 checkpoint
  -> 4a5ac99 QWB RC staging confirmation
  -> aef3e1e Qortium publishing/write-gate skills
```

Every newer branch is an ancestor of `aef3e1e`; there are no divergent
orchestration lines.

---

## 6. Comparison of newer branches against checked-out branches

### 6.1 qortal-dev-workspace

All newer branches are ahead of `main` and 0 behind. None modifies the active
platform pack.

| Branch | Commits ahead | Files changed | Added / modified / deleted | Lines added / deleted | Main content |
| --- | ---: | ---: | --- | --- | --- |
| `owner-runtime-docs-20260913` | 2 | 20 | 14 / 6 / 0 | +2,737 / -95 | Shadow Archives owner-runtime checkpoint, blog/video/contact evidence, project-context update |
| `homepage-test-maintenance-docs-20260915` | 4 | 21 | 15 / 6 / 0 | +2,791 / -95 | Owner-runtime content plus homepage maintenance and phase-closure docs |
| `blog-publish-runtime-handoff-20261001` | 5 | 23 | 17 / 6 / 0 | +3,260 / -95 | Owner-runtime content plus blog remediation, license/README audit, visual correction |
| `phase-4-checkpoint-docs-20260916` (QWB) | 3 | 177 | 177 / 0 / 0 | +13,320 / 0 | QWB audits, phase 0–4 implementations, visual/runtime evidence, RC validation, project context |
| `documentation-backfill-20261001` (QWB) | 1 | 145 | 145 / 0 / 0 | +11,255 / 0 | QWB documentation backfill with packed evidence and updated project context |

**Important:** Across every newer Qortal branch, the changed paths are confined
to `docs/<project-slug>/` and `projects/<project-slug>.md`. No newer branch
changes `agents/`, `docs/architecture/`, `docs/workflows/`, `templates/`, or
`tools/`.

### 6.2 QWB branch comparison

The two QWB branches share 144 added files but are not identical:

- `694dff0` (phase-4 checkpoint): 177 files, 3 commits, expanded evidence,
  including the unpacked RC staging-confirmation directory.
- `18d04c4` (documentation backfill): 145 files, 1 commit, a curated variant
  that replaces the unpacked RC evidence with a ZIP and updates the QWB project
  context.
- Compared with `694dff0`, `18d04c4` adds the checkpoint evidence ZIP, modifies
  the phase-4 checkpoint document and project context, and removes the unpacked
  RC evidence directory.
- Neither branch is a superset of the other. Both must be preserved if all QWB
  evidence is required.

### 6.3 ai-development-orchestration

All branches are linear and additive relative to the checked-out branch.

| Branch | Commits ahead | Files changed | Added / modified / deleted | Lines added / deleted | Skills at tip | Task files | Main content |
| --- | ---: | ---: | --- | --- | ---: | ---: | --- |
| `bootstrap` | 0 | 0 | 0 / 0 / 0 | 0 / 0 | 5 | 0 | Current baseline |
| `owner-runtime-handoff-20260913` | 1 | 4 | 4 / 0 / 0 | +116 / 0 | 5 | 4 | Shadow Archives task artifacts |
| `qortal-skill-promotion-20260914` | 2 | 15 | 12 / 3 / 0 | +1,172 / -61 | 9 | 8 | Qortal skill promotion and tasks |
| `skills-quality-gate-20260914` | 3 | 19 | 13 / 6 / 0 | +1,581 / -83 | 9 | 8 | Skill validator and quality gate |
| `governance-checkpoint-20260914` | 5 | 28 | 19 / 9 / 0 | +1,814 / -110 | 10 | 12 | Governance, identity, report template, Contact skill |
| `qwb-phase-4-checkpoint-20260916` | 6 | 41 | 31 / 10 / 0 | +3,353 / -111 | 13 | 21 | All core Qortal skills, QWB task records, registry update |
| `qwb-rc-7fd03fc-staging-confirmation-20260916` | 7 | 44 | 34 / 10 / 0 | +3,657 / -111 | 13 | 24 | QWB release-candidate confirmation task |
| `qdn-grouped-publish-outcome-20261001` | 9 | 46 | 36 / 10 / 0 | +4,210 / -111 | 15 | 24 | Qortium Home 2 publishing, Qortium write gate, QDN CRUD update, workflow cadence |

### 6.4 Orchestration template and validator additions

At `aef3e1e`:

- `templates/TASK.md`
- `templates/STATUS.json`
- `templates/HANDOFF.md`
- `templates/REPORT.md`
- `tools/validate_skills.py`

The validator is dependency-free Python and checks skill frontmatter, required
headings, platform/maturity boundaries, relative links, duplicate detection,
index coverage, and freshness-date rules. It was inspected but not executed
during Phase 0, to keep this phase strictly preservation-only.

---

## 7. Focused review of newer material

### 7.1 Newer orchestration skills

At `aef3e1e`, the active skill library contains 15 skills:

**Qortal (9)**

- `qortal/bridge-fetch-qdn-resource-normalization` — verified-runtime
- `qortal/cross-app-video-publishing` — verified-runtime
- `qortal/inline-owner-editing` — verified-runtime
- `qortal/private-chat-contact-form` — verified-runtime
- `qortal/qdn-content-crud` — verified-runtime
- `qortal/qdn-derived-index-coherence` — verified-runtime
- `qortal/quitter-announcement` — verified-runtime
- `qortal/registered-name-owner-mode` — verified-runtime
- `qortal/subwire-article-publishing` — verified-runtime

**Qortium (4)**

- `qortium/account-write-gate` — verified-runtime
- `qortium/home2-qdn-publishing` — verified-runtime
- `qortium/qdn-media-readiness` — verified-reference
- `qortium/qdn-resource-discovery` — verified-reference

**Shared (2)**

- `shared/app-bootstrap` — verified-reference
- `shared/capability-harvest` — verified-runtime

The later branches also add a skill index, platform-boundary rules, maturity
states, freshness gates, and the `capability-harvest` loop.

### 7.2 Newer orchestration governance

The later branches change `AGENTS.md`, `WORKFLOW.md`, `GIT-AND-HANDOFF.md`, and
`PROJECT-REGISTRY.md`. Key additions:

- Explicit rule that the CLI/tool profile is not the executing agent.
- `templates/REPORT.md` as the canonical platform report shape.
- Designated orchestration controller rather than hard-wiring Codex Local.
- Bounded-fix versus checkpoint validation cadence.
- Served-bundle hash verification at release gates.
- QWB project registry row.
- Task artifact structure under `tasks/<project>/<task>/`.

### 7.3 Newer Qortal platform material

The Qortal platform pack is stable at `b8fd2f6`. Newer Qortal branches add:

- Shadow Archives owner-runtime, homepage maintenance, blog/video/contact,
  license, and visual-correction reports.
- QWB audits, phase 0–4 implementation reports, visual evidence, runtime
  evidence, and release-candidate validation.
- Updated project contexts for Shadow Archives and QWB.
- Live-evidence JSON and screenshots/artifacts.

These are valuable project evidence. They should not be promoted into shared
platform guidance without a separate review and capability-harvest decision.

---

## 8. Unmerged work that must be preserved

### 8.1 qortal-dev-workspace

All five newer remote branches are unmerged relative to `main`. Preserve every
branch ref and its tip:

| Preserve | Tip | Why |
| --- | --- | --- |
| Shadow Archives owner-runtime | `72b3c4d0e8c8018bde219b4940a2da6202c6b986` | Owner-runtime checkpoint; base of the two newer SA lines |
| Shadow Archives homepage maintenance | `6f5cf53a55b454dba67b0f4fe78d7e44e75b1187` | Homepage maintenance and phase closure; not contained in the blog branch |
| Shadow Archives blog/runtime | `44cf3a5277f2de431b4d09ceae624c9ea8d3791b` | Blog remediation, license/README audit, visual correction |
| QWB phase-4 checkpoint | `694dff049707820ab3a8440692dabfc789f251ab` | Most complete unpacked QWB evidence |
| QWB documentation backfill | `18d04c4102a276b67cec02aaf1450c8134d64f54` | Curated QWB docs/ZIP and updated project context |

No single newer Qortal branch contains both the Shadow Archives and QWB lines.
All five refs are therefore preservation inputs.

### 8.2 ai-development-orchestration

All seven newer branches are unmerged relative to the checked-out branch, but
every one is an ancestor of:

```text
aef3e1ed45c66ef2382e5cc794665d1720f7ad77
```

That tip contains all unique commits and files from the other branches. The
essential preservation target is `aef3e1e`. Keeping the earlier branch refs is
still recommended for audit history and recoverability, but no separate content
integration is required for them.

---

## 9. Recommended migration bases

### 9.1 ai-development-orchestration

**Recommended base:**

```text
origin/agent/shadow-archives/qdn-grouped-publish-outcome-20261001
aef3e1ed45c66ef2382e5cc794665d1720f7ad77
```

Reasons:

1. It is a true descendant of every other remote branch; all seven newer branch
   tips are ancestors.
2. It contains the complete 15-skill library, including the Qortal skill
   promotion, QWB skills, Qortium Home 2 publishing, and Qortium account-write
   gate.
3. It contains `tools/validate_skills.py`, the skill index, and the quality gate.
4. It contains the four templates (`TASK`, `STATUS`, `HANDOFF`, `REPORT`).
5. It contains the 24 task artifacts from the QWB and Shadow Archives work.
6. It contains the latest governance changes, including executing-agent
   identity, controller semantics, bounded-fix cadence, and release-gate
   bundle-hash verification.
7. No newer work exists outside this tip.

This is the cleanest and most complete migration base in either repository.

### 9.2 qortal-dev-workspace

**Recommended repository-wide base:**

```text
main
b8fd2f6fbfcff88c0ab9bc8fd116d5548679d2d2
```

Reasons:

1. Every newer branch descends from `b8fd2f6`; it is the common base.
2. No newer branch is a superset of the others.
3. The active Qortal platform pack is unchanged after `b8fd2f6`; the newer
   branches contain project evidence and project contexts only.
4. Selecting `main` avoids making one divergent evidence line the accidental
   base of the platform pack.

**Required preservation inputs for later integration:**

- Shadow Archives evidence: `44cf3a5` and `6f5cf53` (the latter supplies the
  homepage maintenance line; both include the owner-runtime base).
- QWB evidence: `694dff0` for unpacked runtime evidence and/or `18d04c4` for the
  curated documentation/ZIP variant.
- `72b3c4d` may be kept as the owner-runtime ancestor but is contained in both
  newer Shadow Archives lines.

If the consolidation only needs the platform pack, `b8fd2f6` is sufficient. If
it needs the latest project contexts and evidence, the named tips must be
carried forward separately.

---

## 10. Risks and constraints carried into the next phase

1. **Do not select a Qortal evidence branch as the sole base.** It would omit
   either the other Shadow Archives line or the QWB line.
2. **Do not treat the QWB branches as duplicates.** They differ in evidence
   packaging and project-context content.
3. **Do not use the current local orchestration branch as the migration base.**
   It is the bootstrap branch and lacks 10 of the 15 skills, the validator,
   the task artifacts, and the newer governance rules.
4. **Do not copy skill material into the platform pack without deduplication.**
   The newer Qortal skills overlap with Qortal platform guidance and need the
   precedence/deduplication work planned for later phases.
5. **Do not expose embedded evidence details.** The historical evidence
   contains operational environment details and public wallet/resource data.
   Reports should cite paths and conclusions, not reproduce credentials,
   private infrastructure, or secret material.
6. **Do not modify or prune branches.** The backups and remote refs are the
   preservation boundary for Phase 0.
7. **Do not run the skill validator against a non-checked-out branch in place.**
   A later phase should use an isolated worktree or extracted archive if
   validation of the migration base is required.

---

## 11. Phase 0 conclusion

Phase 0 is complete:

- Both working trees were verified clean.
- Both current branches and HEADs were recorded and remained unchanged.
- Complete backups including `.git` exist under the requested directory.
- Both backups passed Git integrity and tracked-file verification.
- Remote branch metadata was fetched/verified; local and live remote refs match.
- All local and remote branches, tips, dates, ahead/behind relationships, and
  unique changes were inventoried.
- Newer skills, validator, templates, task artifacts, and Qortal project
  material were reviewed.
- Migration bases were recommended with exact SHAs.
- All unmerged work is identified and preserved.

No tracked files were modified, no branch was changed, no merge/rebase/reset/
checkout/switch occurred, no branch or file was deleted, no commit or push was
made, and no application project, Qortium service, node configuration, or node
data was touched.

Report saved:
`/home/iffi/dev-workspace/docs/workspace/audit/AGENT-WORKSPACE-PHASE0.md`
