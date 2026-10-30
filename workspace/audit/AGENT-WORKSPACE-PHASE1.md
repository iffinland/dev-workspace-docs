# Agent Workspace Consolidation — Phase 1

Date: 2026-10-30
Phase: 1 — establish the canonical agent control plane
Status: complete for Phase 1; no commit or push

Canonical control plane:

```text
/home/iffi/dev-workspace/control-plane
```

Source baseline:

```text
Repository: /home/iffi/dev-workspace/projects/ai-development-orchestration
Remote:     https://github.com/iffinland/ai-development-orchestration.git
Branch:     origin/agent/shadow-archives/qdn-grouped-publish-outcome-20261001
Commit:     aef3e1ed45c66ef2382e5cc794665d1720f7ad77
Tree:       d699f691edaab47b22c9171e3958e99b8d6e594d
```

Phase 0 report used as the migration baseline:

```text
/home/iffi/dev-workspace/docs/workspace/audit/AGENT-WORKSPACE-PHASE0.md
```

---

## 1. Result

A new canonical control plane now exists at
`/home/iffi/dev-workspace/control-plane` on local branch
`phase1/control-plane`, based exactly on the selected orchestration commit.

It contains:

- the shared agent contract, workflow, role definitions and Git/handoff rules;
- task, status, handoff and report templates;
- the validated 15-skill library, index and skill validator;
- separate Qortal and Qortium platform boundaries;
- verified project and environment registries using actual VPS paths;
- OpenCode and DeepSeek adapter notes;
- migration and provenance documentation;
- a control-plane link/structure validator.

Full Qortal and Qortium platform documentation was deliberately **not**
imported. No application project, Qortium runtime, node configuration, service,
database, wallet or original repository was modified.

---

## 2. Source verification and clone method

Before creating the workspace, the selected commit was verified in the original
orchestration repository:

| Check | Result |
| --- | --- |
| Commit exists | Yes |
| Commit type | `commit` |
| Commit SHA | `aef3e1ed45c66ef2382e5cc794665d1720f7ad77` |
| Commit date | 2026-10-01T20:49:08+03:00 |
| Subject | `Add Qortium publishing and write-gate skills` |
| Commit tree | `d699f691edaab47b22c9171e3958e99b8d6e594d` |
| Remote branch SHA | Matches the selected commit |

The new workspace was created with a separate single-branch clone from the
GitHub remote, not by changing the original local checkout:

```bash
git clone --branch agent/shadow-archives/qdn-grouped-publish-outcome-20261001 \
  --single-branch https://github.com/iffinland/ai-development-orchestration.git \
  /home/iffi/dev-workspace/control-plane
```

A new local branch was then created at the exact commit:

```text
phase1/control-plane -> aef3e1ed45c66ef2382e5cc794665d1720f7ad77
```

---

## 3. Established structure

```text
control-plane/
├── AGENTS.md                 canonical contract, precedence and inheritance
├── README.md                 orientation, structure and current baseline
├── WORKFLOW.md               execution and validation workflow
├── GIT-AND-HANDOFF.md        branch, identity, authorization and handoff rules
├── PROJECT-REGISTRY.md       verified project routing
├── ENVIRONMENT.md            non-secret environment and reference registry
├── roles/                    ChatGPT, Codex and DeepSeek role definitions
├── templates/
│   ├── TASK.md
│   ├── STATUS.json
│   ├── HANDOFF.md
│   └── REPORT.md
├── skills/
│   ├── README.md             skill index and lifecycle
│   ├── _template/SKILL.md
│   ├── shared/               2 skills
│   ├── qortal/               9 skills
│   └── qortium/              4 skills
├── platforms/
│   ├── README.md             platform boundary rules
│   ├── qortal/README.md      Qortal boundary and source reference
│   └── qortium/README.md     Qortium boundary and source reference
├── adapters/
│   ├── opencode/README.md    discovery, precedence and restart requirement
│   └── deepseek/README.md    client and credential boundary
├── tools/
│   ├── validate_skills.py    imported validated skill validator
│   └── validate_control_plane.py  new link/structure validator
├── tasks/                    imported historical task artifacts
├── reports/                  imported historical reports
├── research/                 imported historical research
└── docs/migration/
    └── 2026-10-30-phase1-control-plane.md
```

---

## 4. Changes made in the new control plane

### 4.1 Imported unchanged

The validated orchestration material was reused rather than rewritten:

- `WORKFLOW.md`
- `GIT-AND-HANDOFF.md` (one current-path correction)
- `roles/`
- `skills/` (15 active skills, index, template)
- `tools/validate_skills.py`
- `templates/`
- `reports/`
- `tasks/`
- `research/`
- `CLAUDE.md`

Historical reports and task artifacts retain their original paths as provenance.
They are not current routing.

### 4.2 Modified active files

| File | Change |
| --- | --- |
| `AGENTS.md` | Added canonical workspace, OpenCode/DeepSeek discovery, instruction precedence, platform-pack routing and Phase 1 boundary notes |
| `README.md` | Replaced the bootstrap README with control-plane orientation, baseline, structure, clients and platform boundary |
| `PROJECT-REGISTRY.md` | Replaced old workstation paths with verified VPS project paths; marked unverified legacy projects as deferred |
| `ENVIRONMENT.md` | Replaced old paths and dated node assertions with a non-secret current-path registry and task-time freshness gate |
| `GIT-AND-HANDOFF.md` | Corrected the worktree root to `/home/iffi/dev-workspace/projects` |
| `templates/TASK.md` | Corrected the contract path to the new control plane |
| `skills/README.md` | Added platform-pack links, source precedence and historical-evidence note |
| Four Qortal skill files | Replaced old Qortal evidence root with the actual VPS path and noted preserved-branch evidence |

The four updated skill files are:

- `skills/qortal/cross-app-video-publishing/SKILL.md`
- `skills/qortal/qdn-derived-index-coherence/SKILL.md`
- `skills/qortal/quitter-announcement/SKILL.md`
- `skills/qortal/subwire-article-publishing/SKILL.md`

### 4.3 New files

| File | Purpose |
| --- | --- |
| `platforms/README.md` | Platform-pack boundary rules |
| `platforms/qortal/README.md` | Qortal boundary, source repo, baseline and preserved evidence branches |
| `platforms/qortium/README.md` | Qortium boundary, local reference repositories and deferred import |
| `adapters/opencode/README.md` | OpenCode discovery, precedence and restart note |
| `adapters/deepseek/README.md` | DeepSeek client boundary and no-secret rule |
| `docs/migration/2026-10-30-phase1-control-plane.md` | Exact migration and provenance record |
| `tools/validate_control_plane.py` | Active Markdown link and required-file validator |

---

## 5. Path correction policy

Active control-plane files now use the actual VPS layout:

```text
/home/iffi/dev-workspace/control-plane
/home/iffi/dev-workspace/projects
/home/iffi/dev-workspace/backups/agent-workspaces-20261030
```

Old `/home/iffi/VsCodec-Projects/...` and uppercase `AI-Orchestration` paths
remain only inside historical reports, task artifacts and the migration record
where they describe provenance. They are not current routing and must not be
used to locate active files.

This distinction is intentional: rewriting historical reports would falsify
their recorded evidence.

---

## 6. OpenCode integration

The owner's global OpenCode configuration was updated minimally:

```text
~/.config/opencode/opencode.jsonc
```

It now includes:

```json
"instructions": [
  "/home/iffi/dev-workspace/control-plane/AGENTS.md"
]
```

Verification from a registered project (`/home/iffi/dev-workspace/projects/Blogs`):

```bash
opencode debug config
```

showed the resolved instruction path:

```text
"instructions": [
  "/home/iffi/dev-workspace/control-plane/AGENTS.md"
]
```

The root instruction file is readable at the configured path. This means
OpenCode can load the canonical root contract when its working directory is a
registered project outside the control-plane directory.

Existing project-level `AGENTS.md` files were not overwritten. The precedence
rule is defined in the root contract:

1. owner task authorization and constraints;
2. control-plane safety, authority and handoff rules;
3. platform pack;
4. project-level `AGENTS.md` deltas;
5. matching skills after freshness check;
6. historical evidence and reports.

Current platform source/runtime remains the authority for platform behavior.

The global OpenCode configuration is loaded at startup. **Restart OpenCode**
for the new instruction path to take effect in a running session.

The original global config content was preserved at:

```text
/home/iffi/dev-workspace/backups/agent-workspaces-20261030/opencode-global-config.pre-phase1.jsonc
```

---

## 7. DeepSeek integration boundary

No DeepSeek API key, token, endpoint credential or client configuration was
copied into the control plane. DeepSeek is documented as a client/execution
surface that reads the same canonical files.

No DeepSeek client installation, configuration, reachability or automated
dispatch was verified in Phase 1. That remains a separate future step.

---

## 8. Platform separation

Qortal and Qortium are separate platform packs:

- Qortal pack references the preserved Qortal source at
  `/home/iffi/dev-workspace/projects/qortal-dev-workspace` and the five
  preserved evidence branch tips from Phase 0.
- Qortium pack references the local Home/Core checkouts and explicitly does not
  assume the old Qortium governance workspace exists on this VPS.
- Full platform documentation is not imported yet.
- No Qortal contract is applied to Qortium, or the reverse.

Skill separation validation:

```text
qortal: 9
qortium: 4
shared: 2
platform/directory mismatches: 0
```

---

## 9. Preservation of originals

The two original repositories remain unchanged:

| Repository | Branch | HEAD | Working tree | Backup HEAD match |
| --- | --- | --- | --- | --- |
| `qortal-dev-workspace` | `main` | `b8fd2f6fbfcff88c0ab9bc8fd116d5548679d2d2` | clean | Yes |
| `ai-development-orchestration` | `agent/orchestration/bootstrap` | `c52e04841adfde062e579463a39bf0463609c8d5` | clean | Yes |

Phase 0 backups remain at:

```text
/home/iffi/dev-workspace/backups/agent-workspaces-20261030/
```

No original branch was switched, merged, reset, rebased or deleted. No original
remote branch reference was removed. No application project, Qortium service,
node configuration, database, wallet or runtime data was modified.

---

## 10. Validation results

| Check | Result |
| --- | --- |
| Control-plane HEAD equals selected SHA | PASS — `aef3e1ed45c66ef2382e5cc794665d1720f7ad77` |
| Imported skill validator | PASS — 15 active skills indexed exactly once |
| Control-plane validator | PASS — 37 active Markdown files, 74 relative links, 20 required files |
| All Markdown link scan | PASS — 79 relative links, 0 broken |
| Platform separation | PASS — 9 Qortal, 4 Qortium, 2 shared, 0 directory mismatches |
| Template presence | PASS — `TASK`, `STATUS`, `HANDOFF`, `REPORT` present |
| Common secret signatures | PASS — no private-key or common token signatures found |
| Credential files | PASS — no `.env`, key, token or credential files copied |
| OpenCode instruction resolution | PASS — global config resolves the control-plane `AGENTS.md` |
| OpenCode root file readable | PASS |
| `git diff --check` | PASS |
| Original repository working trees | PASS — both clean |
| Original HEADs match Phase 0 backups | PASS |

The secret scan is conservative and is not a formal guarantee. No secrets were
intentionally copied, and no wallet, credential or private infrastructure data
is reproduced in this report.

---

## 11. Exact Git state of the new control plane

```text
Path:   /home/iffi/dev-workspace/control-plane
Branch: phase1/control-plane
HEAD:   aef3e1ed45c66ef2382e5cc794665d1720f7ad77
Remote: https://github.com/iffinland/ai-development-orchestration.git
Commit: none created in Phase 1
Push:   none
```

Working-tree state includes only the Phase 1 changes:

```text
 M AGENTS.md
 M ENVIRONMENT.md
 M GIT-AND-HANDOFF.md
 M PROJECT-REGISTRY.md
 M README.md
 M skills/README.md
 M skills/qortal/cross-app-video-publishing/SKILL.md
 M skills/qortal/qdn-derived-index-coherence/SKILL.md
 M skills/qortal/quitter-announcement/SKILL.md
 M skills/qortal/subwire-article-publishing/SKILL.md
 M templates/TASK.md
?? adapters/
?? docs/
?? platforms/
?? tools/validate_control_plane.py
```

No tracked file was committed, staged, pushed, merged, reset, rebased or deleted.

---

## 12. Unresolved issues and deferred work

1. **Full platform documentation is not imported.** Qortal and Qortium packs are
   boundaries only. Import belongs to a later phase.
2. **Project contexts are not imported.** The registry routes to verified project
   paths; canonical context remains pending.
3. **Qortium platform governance source is unresolved.** No canonical Qortium
   workspace was verified on this VPS. Home/Core are reference checkouts only.
4. **Historical paths remain in imported historical artifacts.** This is
   deliberate provenance, not current routing.
5. **Some Qortal evidence lives on preserved branches.** Skills now note this;
   verify the branch and file before citing.
6. **OpenCode restart required.** The running session keeps the old config until
   restart.
7. **DeepSeek client was not verified.** No API credential or client deployment
   was performed.
8. **No live platform, node, host or runtime validation was performed** because
   Phase 1 is an instruction and structure migration, not a platform task.
9. **No automation was introduced.** There is no background agent, polling,
   scheduler, automatic dispatch, automatic commit/push or publish behavior.

---

## 13. Phase 1 conclusion

Phase 1 is complete.

The canonical control plane exists at the requested path, is based exactly on
the selected orchestration commit, has separate Qortal/Qortium boundary packs,
uses current VPS paths in active guidance, preserves validated material, and
defines clear OpenCode/DeepSeek inheritance without overwriting project-level
instructions.

Phase 2 has not been started. Full Qortal/Qortium platform documentation import,
project-context migration and consolidation remain future work.

Report saved:
`/home/iffi/dev-workspace/docs/workspace/audit/AGENT-WORKSPACE-PHASE1.md`
