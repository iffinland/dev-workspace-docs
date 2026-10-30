# Phase 1.6 — Canonical project-based report storage

Date: 2026-10-30
Status: PASS
Executing agent: unknown (not established by evidence in this session; recorded as unknown rather than inferred from the CLI/profile)
Scope: documentation, governance and report storage only
Authorized actions: local documentation edits and file moves. No application source, Git branch, commit, push, build, deployment, runtime, node, wallet or secret action.

## 1. Objective

Implement a permanent project-based documentation and reporting structure under:

```text
/home/iffi/dev-workspace/docs/
```

Every registered project receives its own central documentation root. Reports are
grouped by category. Cross-project, control-plane and general workspace material
goes under `workspace/`. Application technical documentation stays in the project
repository.

## 2. Method

1. Inspected the existing `docs/` contents and the control-plane governance set.
2. Defined the canonical directory structure and category semantics.
3. Created the workspace categories needed for existing artifacts.
4. Created one central root per registered project with a routing README.
5. Moved existing workspace-level reports into `workspace/audit/` and the
   environment document into `workspace/reports/`, preserving contents.
6. Updated every reference and self-path that pointed at a moved file.
7. Added the canonical policy document and wired it into AGENTS, workflow,
   handoff, registry, environment, template, role and adapter guidance.
8. Updated project-level report routing to each project's central root.
9. Ran the existing validators plus a Phase 1.6 structural/link/path validator.

## 3. Directory architecture implemented

```text
/home/iffi/dev-workspace/docs/
├── workspace/
│   ├── README.md
│   ├── audit/          (populated)
│   └── reports/        (populated)
├── Blogs/
│   └── README.md
├── nodefm-station/
│   └── README.md
├── Qortium-United-Community/
│   └── README.md
├── shadow-archives-webportal-QORTAL/
│   └── README.md
├── Qortal-Video-Bridge/
│   └── README.md
└── wordpress-to-qortal/
    └── README.md
```

### Empty-directory rule

The task requires the six categories (`audit/`, `reports/`, `research/`,
`implementation/`, `validation/`, `handoffs/`) for each project but also
forbids creating empty directories for projects without documentation yet.

Implemented interpretation:

- Each registered project has a real central root containing a `README.md` that
  explains its six categories and routing policy.
- Category directories are created when the first artifact of that category is
  added.
- No empty category directory was created.
- `workspace/audit/` and `workspace/reports/` exist because they contain the
  migrated artifacts.
- No regular file is present directly in the `docs/` root.

Validation confirmed there are no empty directories under `docs/`.

## 4. Report movement (content preserved)

| Previous path | New path | Category rationale |
| --- | --- | --- |
| `docs/AGENT-WORKSPACE-COMPARISON.md` | `docs/workspace/audit/AGENT-WORKSPACE-COMPARISON.md` | Comparative architecture audit |
| `docs/AGENT-WORKSPACE-PHASE0.md` | `docs/workspace/audit/AGENT-WORKSPACE-PHASE0.md` | Workspace audit |
| `docs/AGENT-WORKSPACE-PHASE1.md` | `docs/workspace/audit/AGENT-WORKSPACE-PHASE1.md` | Workspace audit |
| `docs/AGENT-WORKSPACE-PHASE1.5.md` | `docs/workspace/audit/AGENT-WORKSPACE-PHASE1.5.md` | Workspace audit |
| `docs/DEVELOPMENT-ENVIRONMENT.md` | `docs/workspace/reports/DEVELOPMENT-ENVIRONMENT.md` | Environment/infrastructure report |

Contents were preserved. Only self-referential paths inside the moved documents
were corrected so they no longer point at the old root locations.

No project-specific report existed under `docs/` at migration time, so no
project report was moved into `workspace/`. No historical report inside a
preserved reference repository or control-plane task archive was moved.

## 5. Reference and link updates

| File | Update |
| --- | --- |
| `control-plane/platforms/qortal/README.md` | Phase 0 comparison reference now points to `docs/workspace/audit/AGENT-WORKSPACE-PHASE0.md` |
| `docs/workspace/audit/AGENT-WORKSPACE-PHASE0.md` | Self report path updated |
| `docs/workspace/audit/AGENT-WORKSPACE-PHASE1.md` | Phase 0 reference and self path updated |
| `docs/workspace/audit/AGENT-WORKSPACE-PHASE1.5.md` | Self report path updated |
| `docs/workspace/audit/AGENT-WORKSPACE-COMPARISON.md` | Self report path updated |

A workspace-wide search now returns no active reference to
`/home/iffi/dev-workspace/docs/AGENT-WORKSPACE-*.md` or
`/home/iffi/dev-workspace/docs/DEVELOPMENT-ENVIRONMENT.md`.

## 6. Governance policy introduced

Canonical policy:

```text
/home/iffi/dev-workspace/control-plane/docs/REPORT-STORAGE-POLICY.md
```

The policy defines:

- the canonical root and required structure;
- the project-to-root mapping;
- the six category definitions;
- storage, naming and link rules;
- the boundary between compact control-plane task artifacts and central reports;
- preservation rules for historical material;
- the Phase 1.6 migration record;
- enforcement and validation requirements.

Policy references were added to:

| File | Change |
| --- | --- |
| `control-plane/AGENTS.md` | New “Report storage” section; platform bullet now points to the central policy |
| `control-plane/WORKFLOW.md` | Handoff/completion now requires canonical project/category report placement |
| `control-plane/GIT-AND-HANDOFF.md` | Task artifacts distinguished from detailed central reports; report-location paragraph updated |
| `control-plane/templates/REPORT.md` | Added category/project fields, canonical path guidance and path rules |
| `control-plane/PROJECT-REGISTRY.md` | Added a central report root column for every registered project |
| `control-plane/README.md` | Added policy to structure and main documents |
| `control-plane/ENVIRONMENT.md` | Added the `central-docs` workspace path |
| `control-plane/roles/DEEPSEEK.md` | DeepSeek must store reports per policy and keep chat output short |
| `control-plane/adapters/opencode/README.md` | OpenCode report-storage boundary documented |
| `control-plane/platforms/qortal/README.md` | Moved-report link corrected |

## 7. Project report routing

| Project ID | Central report root | Project instructions updated |
| --- | --- | --- |
| `qortium/blogs` | `/home/iffi/dev-workspace/docs/Blogs/` | `AGENTS.md`, `README.md` |
| `qortium/nodefm-station` | `/home/iffi/dev-workspace/docs/nodefm-station/` | `AGENTS.md`, `README.md`, `docs/ROADMAP.md` |
| `qortium/qortium-united-community` | `/home/iffi/dev-workspace/docs/Qortium-United-Community/` | `AGENTS.md`, `README.md` |
| `qortal/shadow-archives-webportal` | `/home/iffi/dev-workspace/docs/shadow-archives-webportal-QORTAL/` | `AGENTS.md` |
| `qortal/qortal-video-bridge` | `/home/iffi/dev-workspace/docs/Qortal-Video-Bridge/` | No project `AGENTS.md`; inherits the control plane |
| `qortal/wordpress-to-qortal` | `/home/iffi/dev-workspace/docs/wordpress-to-qortal/` | No project `AGENTS.md`; inherits the control plane |

## 8. Exact files changed in Phase 1.6

### New central documentation files

- `docs/workspace/README.md`
- `docs/workspace/audit/PHASE1.6-REPORT-STORAGE-MIGRATION.md` (this report)
- `docs/Blogs/README.md`
- `docs/nodefm-station/README.md`
- `docs/Qortium-United-Community/README.md`
- `docs/shadow-archives-webportal-QORTAL/README.md`
- `docs/Qortal-Video-Bridge/README.md`
- `docs/wordpress-to-qortal/README.md`

### Moved files

- `docs/AGENT-WORKSPACE-COMPARISON.md` → `docs/workspace/audit/AGENT-WORKSPACE-COMPARISON.md`
- `docs/AGENT-WORKSPACE-PHASE0.md` → `docs/workspace/audit/AGENT-WORKSPACE-PHASE0.md`
- `docs/AGENT-WORKSPACE-PHASE1.md` → `docs/workspace/audit/AGENT-WORKSPACE-PHASE1.md`
- `docs/AGENT-WORKSPACE-PHASE1.5.md` → `docs/workspace/audit/AGENT-WORKSPACE-PHASE1.5.md`
- `docs/DEVELOPMENT-ENVIRONMENT.md` → `docs/workspace/reports/DEVELOPMENT-ENVIRONMENT.md`

### New control-plane policy

- `control-plane/docs/REPORT-STORAGE-POLICY.md`

### Control-plane governance edits

- `control-plane/AGENTS.md`
- `control-plane/WORKFLOW.md`
- `control-plane/GIT-AND-HANDOFF.md`
- `control-plane/templates/REPORT.md`
- `control-plane/PROJECT-REGISTRY.md`
- `control-plane/README.md`
- `control-plane/ENVIRONMENT.md`
- `control-plane/roles/DEEPSEEK.md`
- `control-plane/adapters/opencode/README.md`
- `control-plane/platforms/qortal/README.md`

### Project documentation edits

- `projects/Blogs/AGENTS.md`
- `projects/Blogs/README.md`
- `projects/nodefm-station/AGENTS.md`
- `projects/nodefm-station/README.md`
- `projects/nodefm-station/docs/ROADMAP.md`
- `projects/Qortium-United-Community/AGENTS.md`
- `projects/Qortium-United-Community/README.md`
- `projects/shadow-archives-webportal-QORTAL/AGENTS.md`

No application source code was modified. `Qortal-Video-Bridge` and
`wordpress-to-qortal` repositories were not modified.

## 9. Validation results

| Check | Result | Evidence |
| --- | --- | --- |
| All registered projects have a central root | PASS | 6/6 project roots exist with routing README |
| Directory structure consistent | PASS | `workspace/audit`, `workspace/reports`, project roots; no empty directories |
| No files directly in `docs/` root | PASS | Root contains directories only |
| Existing reports classified and stored correctly | PASS | 4 workspace audits + 1 workspace report |
| Moved contents preserved | PASS | Only self-path references corrected |
| No active stale moved-file references | PASS | Workspace-wide search returned none |
| All updated relative links resolve | PASS | 109/109 active links |
| All active absolute paths exist | PASS | 44/44 paths |
| Registry report roots exist | PASS | 6/6 central roots verified |
| Control-plane validator | PASS | 38 active Markdown files, 85 relative links, 20 required files |
| Skills validator | PASS | 15 active skills indexed exactly once |
| Policy accessible to OpenCode | PASS | OpenCode loads canonical `AGENTS.md`; new Report storage section links the policy |
| No contradictory report-storage instructions found | PASS | Project and control-plane instructions now point to the same policy |
| `git diff --check` (changed repos) | PASS | No whitespace errors |
| Build / commit / push / branch change | NOT PERFORMED | Not authorized |

## 10. Unresolved items and risks

1. The current OpenCode session was started before the governance edits. The
   updated `AGENTS.md`, `WORKFLOW.md` and project instructions take effect in a
   new session/restart. The policy file itself is already on disk.
2. Category subdirectories for the six projects are intentionally not created
   until the first report of each category is written, per the task's
   empty-directory rule. The project root README and the policy document define
   the exact paths.
3. Historical reports in `control-plane/reports/`, `control-plane/tasks/`, the
   migration record and preserved reference repositories still contain old
   workstation paths by design; they were not migrated.
4. Central documentation is not in a Git repository; it is a local VPS evidence
   store. Durable remote publication remains a separate authorized action.
5. No project-specific reports existed in `docs/` during this migration, so the
   project category directories are documented but not yet populated.

## 11. Report metadata

- Canonical report path: `/home/iffi/dev-workspace/docs/workspace/audit/PHASE1.6-REPORT-STORAGE-MIGRATION.md`
- Category: `workspace/audit`
- Executing agent: unknown (not established by evidence)
- Git state: uncommitted; no commit or push
