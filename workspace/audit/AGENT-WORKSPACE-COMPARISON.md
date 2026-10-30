# Agent Workspace Comparison and Consolidation Proposal

Date: 2026-10-30  
Audit type: read-only comparative architecture audit  
Scope:

- `/home/iffi/dev-workspace/projects/qortal-dev-workspace`
- `/home/iffi/dev-workspace/projects/ai-development-orchestration`

Primary objective: determine how the two agent-development environments are
structured, what each provides, where they overlap or conflict, and how they
could be consolidated into one maintainable autonomous development workflow.

This report is an audit and recommendation only. No repository file, application
source, node configuration, service, or running Qortium environment was modified.
The only file created is this report. No setup or migration script was executed,
no package was installed, and no commit, push, merge, or QDN/GitHub mutation was
performed.

---

## 0. Audit baseline and method

### 0.1 Repository baseline

| Item | qortal-dev-workspace | ai-development-orchestration |
| --- | --- | --- |
| Remote | `https://github.com/iffinland/qortal-dev-workspace.git` | `https://github.com/iffinland/ai-development-orchestration.git` |
| Checked-out branch | `main` | `agent/orchestration/bootstrap` |
| Checked-out HEAD | `b8fd2f6` (2026-09-13) | `c52e048` (2026-09-13) |
| Working tree | clean | clean |
| Local commits visible | 13 | 3 |
| Tracked files | 55 (53 Markdown) | 31 (25 Markdown) |
| Approximate size excluding `.git` | 884 KB | 484 KB |
| Newer remote branches | Yes, through 2026-10-01 | Yes, through 2026-10-01 |
| Local HEAD equals remote default | Yes, `origin/main` | Yes, `origin/agent/orchestration/bootstrap` |

Both repositories are documentation and governance repositories. Neither
contains application source, package manifests, dependency lockfiles, CI
workflows, runtime services, or executable agent dispatchers.

### 0.2 Method

The audit read the complete file inventory, root contracts, workflows, role
overlays, templates, skills, selected reports, Git history, local branch state,
cached remote branches, and cross-repository references. Existing remote refs
were inspected without fetching. No network or live-node evidence was collected
for this audit. The repositories' own historical runtime reports were read as
evidence but were not re-validated against a live host.

Two important limitations apply:

1. **Local HEAD is not the newest branch state.** Both repositories have newer
   agent branches containing additional skills, tasks, templates, and reports.
   Those branches were inspected for direction, but the checked-out local trees
   remain the baseline for this comparison.
2. **Path assumptions are out of date for this VPS.** Both repositories were
   written against an earlier workstation layout (`/home/iffi/VsCodec-Projects/...`)
   and an uppercase orchestration path (`AI-Orchestration`). The current clones
   are under `/home/iffi/dev-workspace/projects/...`. This is a configuration
   drift issue, not a Git error.

### 0.3 Summary verdict

The two repositories are **complementary but overlapping**, not competitors.

- `qortal-dev-workspace` is the deeper Qortal platform execution system. Its
  strength is verified, Qortal-specific engineering and validation guidance
  backed by a real project and real runtime evidence.
- `ai-development-orchestration` is the better cross-platform control plane. Its
  strength is the reusable capability model, role separation, project registry,
  handoff state machine, and skill lifecycle.

The recommended target is **one control plane with platform packs**, not a
flattened repository and not an autonomous runtime that is invented before the
governance is stable. The orchestration repository should become the canonical
control plane; the Qortal workspace should become its first complete platform
pack plus project-evidence source. Consolidation should be phased and
non-destructive.

---

## 1. Architecture of each repository

### 1.1 qortal-dev-workspace

**Type:** Qortal-specific governance, execution, and evidence repository.

**Architecture layers:**

```text
README.md / AGENTS.md
  -> agents/                     task routing and Qortal platform guides
       -> 00-SESSION-START.md
       -> 01-TASK-CLASSIFICATION.md
       -> task-class guides
       -> roles/ thin role overlays
  -> docs/architecture/          Qortal dApp development standard
  -> docs/governance/            source-of-truth and lifecycle
  -> docs/workflows/             Workflow v2, DeepSeek model, report policy
  -> docs/<project-slug>/        historical project reports and evidence
  -> projects/                   canonical project context
  -> templates/                  task, audit, implementation, handoff, review
  -> tools/validate-workspace.sh structural validator
```

**Operating model:**

- One shared guide set for ChatGPT, DeepSeek, and Codex.
- Every substantial task starts at `agents/00-SESSION-START.md` and is classified
  through `agents/01-TASK-CLASSIFICATION.md`.
- Workflow v2 requires evidence before implementation, one primary objective,
  non-interchangeable validation layers, adversarial self-audit, and a truthful
  handoff.
- Qortal Core, Qortal Hub, and the current Q-App framework are the platform
  authority. Legacy `qortal-ui` is explicitly excluded.
- Reports are stored under `docs/<project-slug>/` in the workspace.
- The only executable artifact is a structural Markdown/hygiene validator.

**Scope:** Qortal only. It was deliberately derived from an earlier Qortium
workspace but re-verified for Qortal rather than renamed.

**Current project content:** The `shadow-archives-webportal` project context and
its architecture, implementation, validation, release, runtime, audit, and
handoff reports. This is the strongest real-world evidence set in either
repository.

### 1.2 ai-development-orchestration

**Type:** Cross-platform orchestration and reusable-capability repository.

**Architecture layers:**

```text
AGENTS.md / WORKFLOW.md / README.md
  -> roles/                     ChatGPT, Codex, DeepSeek role contracts
  -> PROJECT-REGISTRY.md        project routing by platform and path
  -> ENVIRONMENT.md             node/reference registry and freshness gate
  -> GIT-AND-HANDOFF.md         branch, task, identity, handoff state machine
  -> skills/
       -> shared/               platform-neutral capability
       -> qortal/               Qortal-specific reusable capability
       -> qortium/              Qortium-specific reusable capability
       -> _template/            canonical skill shape
  -> templates/                 TASK, HANDOFF, STATUS
  -> reports/                   historical bootstrap/audit evidence
  -> research/                  reusable-pattern research
  -> CLAUDE.md                  thin future adapter
```

**Operating model:**

- A single shared contract for Qortal and Qortium work.
- Reuse-first execution: search skills, run their freshness gate, reuse proven
  capability, investigate only the delta, validate, then harvest new knowledge.
- Platform source/runtime remains authoritative; skills summarize evidence and
  never override fresher source.
- Explicit authority separation between owner intent, orchestration, platform
  workspaces, application repositories, and historical evidence.
- Git handoff uses task-scoped authorization, exact SHAs, `STATUS.json` states,
  and separate executing-agent and report-writer identities.
- No background polling, notification service, API agent runner, or scheduled
  job is claimed or installed.

**Scope:** Qortal and Qortium, with shared engineering capabilities.

**Newer branch direction:** The remote branches through 2026-10-01 add a much
larger skill library (QDN CRUD, derived-index coherence, owner mode, inline
editing, private-chat contact, SubWire/article publishing, Qortium Home 2
publishing, Qortium account-write gate), a `tasks/<project>/<task>/` artifact
structure, a `REPORT.md` template, and `tools/validate_skills.py`. These are not
present on the checked-out local branch and are not part of the audited local
baseline, but they show the intended direction clearly.

### 1.3 Relationship implied by the repositories

The Qortal repository already routes to a universal contract:

- `qortal-dev-workspace/AGENTS.md` says to read
  `/home/iffi/VsCodec-Projects/AI-Orchestration/AGENTS.md` first.
- `agents/00-SESSION-START.md`, the DeepSeek work model, the report policy, and
  role overlays repeat that universal-coordination instruction.

That intent is correct, but it is only a documentation-level link. The path is
stale for this VPS, the universal repository does not own the Qortal platform
pack, and neither repository actually dispatches or executes an agent. The
integration is therefore **declared, not operational**.

---

## 2. Existing capabilities of each repository

### 2.1 qortal-dev-workspace capabilities

| Capability | Evidence | Assessment |
| --- | --- | --- |
| Qortal-specific task entry and classification | `agents/00-SESSION-START.md`, `agents/01-TASK-CLASSIFICATION.md` | Strong |
| Qortal dApp architecture standard | `docs/architecture/qortal-dapp-development-standard.md` | Strong, source-linked |
| QDN, bridge, host, routing, CSP, dev-mode guidance | `agents/qortal-qdn-and-bridge.md` | Strong, source-linked |
| Identity, authority, data-integrity, rendering safety | `agents/qortal-architecture-and-data-integrity.md` | Strong |
| QDN publication, discovery, N+1, caching, scaling | `agents/qdn-publication-discovery-and-scaling.md` | Strong |
| Runtime diagnostics and performance | `agents/runtime-diagnostics-and-performance.md` | Strong |
| Live node/host/QDN validation layers | `agents/live-qdn-validation.md` | Strong |
| Issue-driven audit and refactor | `agents/issue-driven-audit-and-refactor.md` | Strong |
| Release and provenance | `agents/app-release-and-provenance.md` | Strong |
| Git and generated-file hygiene | `agents/git-generated-files-and-hygiene.md` | Good |
| Report storage and owner handoff | `docs/workflows/report-storage-policy.md`, `agents/final-report-and-owner-handoff.md` | Strong but path-specific |
| Workflow v2 execution contract | `docs/workflows/workflow-v2.md` | Strong |
| DeepSeek primary work model and risk levels | `docs/workflows/deepseek-primary-work-model.md` | Good |
| Thin role overlays | `agents/roles/` | Good, but duplicated |
| Project context template and real project context | `templates/PROJECT-CONTEXT.md`, `projects/shadow-archives-webportal.md` | Strong |
| Task/audit/implementation/handoff/review templates | `templates/` | Strong |
| Structural validator | `tools/validate-workspace.sh` | Useful, structural only |
| Real project evidence | `docs/shadow-archives-webportal/` | Strong historical evidence |
| Source-of-truth and lifecycle governance | `docs/governance/source-of-truth-and-lifecycle.md` | Strong |

### 2.2 ai-development-orchestration capabilities

| Capability | Evidence | Assessment |
| --- | --- | --- |
| Cross-platform orchestration contract | `AGENTS.md` | Strong |
| Shared workflow | `WORKFLOW.md` | Strong |
| Project registry | `PROJECT-REGISTRY.md` | Good idea, stale paths |
| Environment/reference registry | `ENVIRONMENT.md` | Detailed, stale for this VPS |
| Git and handoff state machine | `GIT-AND-HANDOFF.md`, `templates/STATUS.json` | Strong |
| Agent identity rule | `AGENTS.md`, `GIT-AND-HANDOFF.md` | Strong and distinctive |
| Role separation | `roles/` | Strong but overlaps platform overlays |
| Reusable skill library | `skills/` | Core differentiator |
| Skill maturity model | `skills/README.md` | Strong |
| Freshness/compatibility gate | `skills/README.md`, each skill | Strong |
| Capability harvest loop | `skills/shared/capability-harvest/SKILL.md` | Strong |
| Shared app bootstrap | `skills/shared/app-bootstrap/SKILL.md` | Useful |
| Qortal video interoperability skill | `skills/qortal/cross-app-video-publishing/SKILL.md` | Candidate at local HEAD |
| Qortium QDN discovery/media skills | `skills/qortium/` | Verified-reference |
| Task/handoff/status templates | `templates/` | Strong |
| Claude future adapter | `CLAUDE.md` | Good thin-adapter pattern |
| Reusable research/audit reports | `reports/`, `research/` | Useful historical evidence |
| Skill quality gate | Newer remote branches | Strong direction, not on local HEAD |
| Task artifact structure | Newer remote branches | Strong direction, not on local HEAD |

### 2.3 Capability comparison

```text
                          qortal-dev-workspace        ai-development-orchestration
Platform depth            Qortal: very deep           Cross-platform: broad
Execution workflow        Workflow v2                 Shared reuse-first workflow
Reusable capability       Partial (guides)            First-class (skills)
Role model                Platform overlays           Universal roles
Project context           Real project context        Registry only
Evidence reports          Rich Qortal evidence        Bootstrap/audit evidence
Automation                Structural validator        Docs only at local HEAD
Extensibility             Qortal-specific             Platform-neutral
Current path accuracy     Stale absolute paths        Stale absolute paths
Branch freshness          Local HEAD behind branches  Local HEAD behind branches
```

---

## 3. Strengths and weaknesses

### 3.1 qortal-dev-workspace

**Strengths**

1. **Deep, source-verified Qortal expertise.** The bridge, QDN, service,
   identity, CSP, dev-proxy, scaling, and validation guidance is specific and
   evidence-based rather than generic advice.
2. **Strong validation philosophy.** Static checks, local runtime, read-only node
   evidence, real host, live QDN, adversarial review, and owner acceptance are
   explicitly non-interchangeable.
3. **Real project proof.** Shadow Archives provides architecture, implementation,
   runtime, release, and owner-validation evidence that most agent workspaces
   lack.
4. **Clear fact labeling.** Verified, inference, unknown, and owner decision are
   distinguished throughout the standard and governance documents.
5. **Report discipline.** Reports have a canonical root, naming convention, and
   required final-path disclosure.
6. **Useful structural validator.** It checks required files, relative links,
   anchors, stale markers, and hygiene terms. It correctly states that it is not
   semantic proof.

**Weaknesses**

1. **Not operational.** It is a documentation system with no dispatcher, task
   runner, queue, or agent adapter. "Autonomous" work still depends on a human
   or external client starting each agent.
2. **Qortal-only.** It cannot directly govern Qortium projects or shared
   cross-platform capability growth.
3. **Duplicated role prose.** Role overlays repeat material that the universal
   orchestration contract also owns, creating two places to update.
4. **Stale environment assumptions.** Absolute paths point to
   `/home/iffi/VsCodec-Projects/...` and an uppercase `AI-Orchestration` path,
   neither of which matches the current workspace.
5. **Report storage inside the governance repository.** This couples historical
   evidence volume to the control repository and makes report commits compete
   with governance changes.
6. **Local HEAD lags the newer remote branches.** The current checkout does not
   contain later project handoffs, blog/video/contact evidence, or the newest
   Qortal skill material.
7. **Validator scope is narrow.** It validates structure and links, not semantic
   correctness, skill quality, or platform claims.
8. **No explicit integration contract for OpenCode, DeepSeek APIs, GitHub
   automation, or secret management.**
9. **Project context is very large.** `projects/shadow-archives-webportal.md` is
   valuable but mixes durable context, historical state, and long operational
   detail; that reduces routing clarity.

### 3.2 ai-development-orchestration

**Strengths**

1. **Best consolidation abstraction.** It separates governance, roles, project
   registry, reusable capability, platform truth, and handoff state.
2. **Reuse-first model.** The skill library is the most important long-term
   productivity mechanism in either repository.
3. **Skill lifecycle.** Platform scope, maturity, freshness gate, invalidation
   triggers, validation, and harvest are explicit.
4. **Cross-platform boundary.** It prevents Qortal behavior from being assumed
   for Qortium and vice versa.
5. **Strong handoff model.** Exact SHA binding, task-scoped authorization,
   single-writer status, and explicit state transitions are better than ad hoc
   chat handoffs.
6. **Agent identity rule.** It prevents the orchestrator or committing writer
   from being falsely recorded as the executor.
7. **Thin adapters.** The Claude adapter pattern avoids a second knowledge base;
   the same pattern can be used for OpenCode and other clients.
8. **Newer branch direction is strong.** The skill index, quality gate, and task
   artifact structure are the right next layer.

**Weaknesses**

1. **Local HEAD is incomplete.** The checked-out bootstrap branch has only five
   skills and no skill validator, task artifacts, or `REPORT.md`; the remote
   branches contain much more.
2. **Registry and environment paths are stale for this VPS.** The project
   registry, environment registry, and platform links describe an earlier
   workstation layout.
3. **No actual agent integration.** There is no OpenCode configuration, DeepSeek
   API client, GitHub App/Action, dispatcher, queue, scheduler, or notification
   service.
4. **Docs-only at the execution layer.** It describes an orchestration contract
   but cannot itself start, retry, monitor, or stop an agent.
5. **Platform contracts are split across repositories.** The universal
   repository depends on platform workspaces it does not contain, and the local
   Qortal checkout still treats the universal path as external.
6. **Template duplication.** `TASK`/`HANDOFF`/`STATUS` here overlap
   `TASK-CONTROLLER`/`OWNER-HANDOFF` in the Qortal repository.
7. **Skill duplication risk.** As skills grow, Qortal platform guidance can be
   copied into skills instead of cited, creating two authorities.
8. **Environment content contains operational details that age quickly.** Node
   versions, endpoints, reference revisions, and tunnel state should be isolated
   from durable governance.
9. **No secret-management design.** This is good in that no secrets are present,
   but a future DeepSeek/GitHub integration needs an explicit external secret
   boundary.
10. **The branch naming is project-oriented but not yet a single release
    strategy.** Newer branches diverge rather than merge into one maintained
    trunk.

### 3.3 Shared weaknesses

- Both repositories rely on manually maintained Markdown consistency.
- Both use absolute paths that do not match the current VPS.
- Both have newer unmerged agent branches; the local checkouts can appear more
  current than they are.
- Neither has a CI gate that enforces the documented validators.
- Neither has a real task board, issue-to-agent binding, or execution log.
- Neither has a tested end-to-end "dispatch -> implement -> validate -> handoff"
  loop on this VPS.
- Neither contains API credentials, which is correct, but neither defines the
  external credential boundary either.

---

## 4. Duplicated and conflicting components

### 4.1 Duplicated components

| Area | qortal-dev-workspace | ai-development-orchestration | Overlap |
| --- | --- | --- | --- |
| Root agent contract | `AGENTS.md` | `AGENTS.md` | Both define mandatory behavior and authority boundaries |
| Workflow | `docs/workflows/workflow-v2.md` | `WORKFLOW.md` | Same evidence/validation/handoff themes with different structure |
| Role definitions | `agents/roles/{CHATGPT,DEEPSEEK,CODEX}.md` | `roles/{CHATGPT,DEEPSEEK,CODEX}.md` | Same three roles, different wording and scope |
| DeepSeek model | `docs/workflows/deepseek-primary-work-model.md` | `roles/DEEPSEEK.md`, `WORKFLOW.md` | Same primary implementer model |
| Task controller | `templates/TASK-CONTROLLER.md`, `templates/DEEPSEEK-TASK.md` | `templates/TASK.md` | Different shapes for the same bounded-task concept |
| Handoff | `templates/OWNER-HANDOFF.md` | `templates/HANDOFF.md` | Different fields and report assumptions |
| Report policy | `docs/workflows/report-storage-policy.md` | `GIT-AND-HANDOFF.md` handoff report rules | Different canonical locations and disclosure rules |
| Source authority | `docs/governance/source-of-truth-and-lifecycle.md` | `AGENTS.md` authority separation | Same principle, different taxonomies |
| Project routing | `projects/<slug>.md` | `PROJECT-REGISTRY.md` | Context vs registry, with no single routing source |
| Qortal platform knowledge | `agents/qortal-*.md`, `docs/architecture/` | `skills/qortal/*` on newer branches | Guides and skills can state the same contract |
| App bootstrap | Project context and architecture reports | `skills/shared/app-bootstrap/SKILL.md` | Same shell-before-branding philosophy |
| Validation | Workflow v2 validation layers | Workflow validation and skill validation | Same layering, different enforcement |
| Git hygiene | `agents/git-generated-files-and-hygiene.md` | `GIT-AND-HANDOFF.md` | Same preservation and authorization rules |

### 4.2 Conflicts and drift

1. **Canonical root conflict.** The Qortal workspace and universal contract
   refer to `/home/iffi/VsCodec-Projects/AI-Orchestration`, while the actual
   current clone is
   `/home/iffi/dev-workspace/projects/ai-development-orchestration`. The
   repositories are logically linked but physically disconnected for this VPS.

2. **Platform path conflict.** `PROJECT-REGISTRY.md` and `ENVIRONMENT.md` point
   to older `/home/iffi/VsCodec-Projects/...` project/reference locations. The
   current VPS uses `/home/iffi/dev-workspace/projects/...`. A new agent
   following those files would inspect the wrong paths or fail to find context.

3. **Role authority conflict.** The Qortal `AGENTS.md` says the universal
   contract owns shared roles, but Qortal still contains full role overlays and
   a separate DeepSeek work model. The intent is "universal owns roles, platform
   owns execution", but the duplication makes the precedence unclear in
   practice.

4. **Workflow duplication.** Workflow v2 and `WORKFLOW.md` both define session
   start, evidence, implementation, review, handoff, and completion. They are
   compatible in spirit, but they are two update targets and already differ in
   terminology (`Codex Local` vs `Codex`, risk levels vs state machine).

5. **Report-location conflict.** Qortal reports belong under
   `qortal-dev-workspace/docs/<slug>/`; orchestration tasks/reports belong under
   the orchestration repository. There is no single rule that tells an agent
   which repository owns a given artifact when a task spans both.

6. **Template conflict.** `OWNER-HANDOFF.md` (Qortal) and `HANDOFF.md`
   (orchestration) have different fields. `TASK-CONTROLLER.md` and `TASK.md`
   differ. A task can satisfy one template and fail the other.

7. **Status vocabulary conflict.** Qortal defines a report status vocabulary
   (`COMPLETE`, `PASS WITH OWNER VALIDATION REQUIRED`, `CHANGES REQUIRED`,
   `ESCALATE TO CODEX`, `BLOCKED`, `NOT VERIFIED`). Orchestration defines
   machine states (`planned`, `implementing`, `review_pending`,
   `owner_validation_pending`, `complete`, `blocked`). The mapping is documented
   in historical reports but is not enforced by one schema.

8. **Skill/guide authority conflict.** On newer branches, Qortal skills restate
   bridge/QDN contracts that also exist in Qortal platform guides. Both claim
   evidence-based authority. Without an explicit precedence rule, a skill can
   drift from the platform pack.

9. **Branch strategy conflict.** Both repositories use
   `agent/<project>/<task>` branches, but the orchestration repository's newer
   branches are not based on one merged trunk and the Qortal repository's newer
   branches are separate project/documentation lines. There is no declared
   integration branch or release cadence.

10. **Freshness conflict.** The local checkout of each repository is older than
    its own remote branches. Any audit or agent that treats the local tree as
    current is working from a stale baseline.

11. **Evidence ownership conflict.** Qortal has rich project reports inside the
    governance repo; orchestration has bootstrap/audit reports and, on newer
    branches, task artifacts. The same event can be recorded in two places in
    two formats.

12. **Environment drift.** `ENVIRONMENT.md` contains dated node/reference
    snapshots and tunnel assertions from an earlier environment. It is useful
    historical evidence, but it is not a safe current environment registry
    without revalidation.

### 4.3 Complementary components

These should not be treated as duplicates:

- Qortal platform deep guides versus orchestration cross-platform governance.
- Qortal project evidence versus orchestration capability research.
- Qortal structural validator versus orchestration skill quality gate.
- Qortal live-host validation discipline versus orchestration task/handoff state.
- Qortal project context versus orchestration project registry: one holds
  context, the other routes to it.
- Qortal-specific role constraints versus universal role responsibilities: they
  should be a thin overlay on one canonical role table, not two independent role
  systems.

---

## 5. Features worth preserving from each repository

### 5.1 Preserve from qortal-dev-workspace

1. **Qortal dApp Development Standard** and its verified source references.
2. **QDN/bridge/host guide**, including the action approval classification,
   host-mediated authentication, dev proxy, CSP, and routing rules.
3. **Architecture and data-integrity guide**, especially identity, authority,
   operation ordering, fail-closed design, and rendering safety.
4. **QDN publication/discovery/scaling guide** and N+1 mitigation patterns.
5. **Live validation guide** and the seven-layer validation distinction.
6. **Release and provenance guide.**
7. **Task classification and issue-driven audit workflow.**
8. **Project context template**, including identity, QDN services, authority,
   commands, validation, and owner decisions.
9. **Report storage policy**, after generalizing its root and retaining its
   disclosure rule.
10. **Structural validator**, after fixing anchor/reference-link coverage and
    reporting limitations.
11. **Real Shadow Archives project context and evidence**, retained as historical
    evidence and as a reference implementation, not as universal policy.
12. **Fact labeling** (verified, inferred, unknown, owner decision).
13. **Adversarial self-audit and evidence-based disagreement rules.**

### 5.2 Preserve from ai-development-orchestration

1. **Root orchestration contract and authority separation.**
2. **Reuse-first workflow and capability harvest loop.**
3. **Skill format, platform scope, maturity states, and freshness gate.**
4. **Project registry concept**, updated to current paths and made
   machine-verifiable.
5. **Environment registry concept**, separated from durable governance and
   revalidated per session.
6. **Git and handoff state machine**, exact-SHA binding, and task-scoped
   authorization.
7. **Agent identity rule** separating executing agent, report writer, and
   orchestrator.
8. **Single-writer status discipline** and idempotent resume rule.
9. **Thin role overlays** as the model for platform-specific role constraints.
10. **Thin adapter pattern** (`CLAUDE.md` now; OpenCode/other clients later).
11. **Skill quality gate** from newer branches, with index coverage and dated
    freshness enforcement.
12. **Task artifact structure** (`TASK`, `STATUS`, implementation, audit) from
    newer branches.
13. **Cross-platform boundary** that prevents Qortal/Qortium contract leakage.
14. **App bootstrap skill**, provided it cites rather than duplicates platform
    packs.

---

## 6. Recommended target architecture

### 6.1 Design principle

Use **one control plane, multiple platform packs, one capability library, one
task/handoff schema, and thin client adapters**.

Do not create one giant repository containing application source, node data, and
every historical report. Do not keep two independent governance repositories.
Do not build an autonomous runner before a single validated workflow exists.

### 6.2 Recommended repository topology

```text
agent-development-control/                 (canonical control plane)
├── AGENTS.md                              minimal universal contract
├── README.md                              orientation and entry point
├── WORKFLOW.md                            one execution workflow
├── governance/
│   ├── authority-and-source-of-truth.md
│   ├── report-storage-policy.md
│   ├── agent-identity.md
│   └── external-action-authority.md
├── roles/
│   ├── CHATGPT.md                         thin universal role
│   ├── CODEX.md
│   └── DEEPSEEK.md
├── platforms/
│   ├── qortal/
│   │   ├── ROUTER.md                      sessions, task classes, precedence
│   │   ├── standard/                      Qortal dApp standard
│   │   ├── guides/                        QDN, bridge, architecture, release...
│   │   └── projects/                      project contexts for Qortal
│   └── qortium/
│       ├── ROUTER.md
│       └── projects/
├── skills/
│   ├── README.md                          index, maturity, quality gate
│   ├── _template/SKILL.md
│   ├── shared/
│   ├── qortal/
│   └── qortium/
├── projects/
│   └── REGISTRY.md                        ID -> platform -> path -> context
├── environments/
│   └── REGISTRY.md                        dated, non-secret, revalidated
├── templates/
│   ├── TASK.md
│   ├── STATUS.json
│   ├── REPORT.md
│   ├── HANDOFF.md
│   └── PROJECT-CONTEXT.md
├── tasks/                                 created on demand per task
├── tools/
│   ├── validate_control.py or .sh
│   └── validate_skills.py
└── adapters/
    ├── opencode.md                        thin mapping, no copied knowledge
    ├── claude.md                          thin mapping
    └── deepseek.md                        client boundary, no secrets
```

Historical evidence can remain in a separate evidence area or in platform
archives:

```text
evidence/
├── qortal/
│   └── shadow-archives-webportal/
└── orchestration/
```

If evidence must stay in Git, keep it under an explicit evidence root and mark
it historical. Do not mix dated reports with active guidance.

### 6.3 Authority precedence

The unified system should declare one precedence order:

```text
current platform source/runtime
  > platform pack technical contract
  > verified skill at its recorded revision
  > shared workflow/governance
  > historical reports
  > legacy notes
```

Owner product decisions remain authoritative for intent and authorization, and
cannot override verified platform facts. A documented owner decision that
contradicts implementation is an implementation gap to report, not a license to
rewrite platform truth.

### 6.4 Skill model

- One canonical skill library only.
- Every skill declares platform, maturity, date, source revisions, freshness
  gate, validation, failure modes, and non-goals.
- Skills cite platform pack paths and source revisions; they do not copy full
  guides.
- A skill whose contract is already covered by a platform guide should link to
  the guide and add only the reusable delta.
- Maturity promotion requires the evidence defined by the skill.
- `verified-runtime` requires our own real runtime evidence, not only donor
  source.
- A quality gate enforces metadata, index coverage, links, duplicate detection,
  and freshness dates.

### 6.5 Workflow model

Use one workflow with these states:

```text
planned -> implementing -> review_pending -> owner_validation_pending -> complete
                    ^             |
                    |             v
              changes_required <-+
any state -> blocked (with reason, missing evidence, resume condition)
```

Every task has:

- one outcome and exit criterion;
- exact repository, branch, base SHA, and preserved work;
- selected skills and freshness result;
- authorized external actions (default none);
- validation evidence by layer;
- canonical report path and remote evidence;
- executing agent and report writer;
- capability-harvest result.

### 6.6 Integration boundaries

**OpenCode**

- Use root `AGENTS.md` as the primary instruction surface.
- Add a thin `adapters/opencode.md` only when needed; do not copy skills or
  governance into an OpenCode-specific tree.
- OpenCode can run validators and read the same canonical files; it does not
  become a second source of truth.

**DeepSeek**

- Treat DeepSeek as an implementation role/client, not as a governance source.
- Keep API keys and endpoint credentials outside Git and outside task artifacts.
- Resolve credentials from the user's environment or a secret store at run time.
- Record only model/role identity and evidence, never secret values.

**GitHub**

- Continue using branches, PRs, and exact SHAs as durable evidence.
- Use GitHub Actions, if added, only for read-only validators and link/skill
  checks initially.
- Do not treat GitHub as an agent dispatcher. A workflow event is not an
  authorization to run autonomous writes.
- No agent gains commit/push/merge authority merely from repository access.

**Qortal / Qortium**

- Keep platform source and runtime authoritative.
- Keep the environment registry non-secret and dated.
- Read-only node access may support evidence; it does not replace real-host
  validation.
- Do not modify nodes, services, or data as part of orchestration.
- Keep Qortal and Qortium skills and contracts separate even when names look
  similar.

### 6.7 What not to build yet

- A multi-agent autonomous scheduler.
- Background polling or a notification service.
- Automatic issue-to-code dispatch.
- Automatic commit/push/publish.
- A single universal business/API wrapper for all apps.
- A monorepo containing all application source.

The governance and evidence model must be stable first. Automation should wrap a
proven workflow, not substitute for one.

---

## 7. Phased consolidation proposal

### Phase 0 — Freeze, inventory, and decide (no changes)

**Goal:** Establish a safe, non-destructive baseline.

- Freeze both repositories; no branch merges or file moves.
- Record exact HEADs, dirty state, remote refs, and open branches.
- Inventory active guidance, skills, templates, project contexts, and reports.
- Decide the canonical control-plane repository and its owner. Recommendation:
  promote `ai-development-orchestration` as the control plane, or create a
  successor repository from it; do not keep two permanent control planes.
- Decide the evidence-retention location before moving any reports.
- Back up both repositories locally and verify the backup.

**Exit criterion:** Owner-approved target repository, evidence policy, and
rollback plan.

### Phase 1 — Create one canonical control plane

**Goal:** One place for universal governance.

- Establish the unified root `AGENTS.md`, `WORKFLOW.md`, roles, and governance
  files.
- Replace hardcoded `/home/iffi/VsCodec-Projects/...` paths with relative paths
  or a documented workspace-root variable.
- Merge the universal authority, report, identity, and external-action rules.
- Keep platform technical execution rules out of the universal root.
- Add one `STATUS.json` schema and one task state machine.
- Add one `REPORT.md` and one `HANDOFF.md` template.

**Exit criterion:** Every universal rule exists once; no duplicate role,
workflow, or handoff authority remains.

### Phase 2 — Import the Qortal platform pack

**Goal:** Make Qortal the first complete platform pack.

- Move or copy the Qortal standard, routers, guides, and project context into
  `platforms/qortal/`.
- Preserve Git history where practical; otherwise record source revision and
  migration provenance.
- Convert Qortal root `AGENTS.md` into a thin platform router, not a second
  universal contract.
- Keep Shadow Archives reports as evidence, outside active guidance.
- Update all relative links and remove stale absolute paths.
- Run the structural validator and a semantic link/provenance review.
- Keep the old qortal repository read-only or clearly marked as the source until
  the new pack is accepted.

**Exit criterion:** A task can start from the control plane, reach the Qortal
platform pack, and complete without reading the old repository.

### Phase 3 — Consolidate skills and resolve duplication

**Goal:** One capability library with no competing authority.

- Import the newest orchestration-branch skills; do not import stale local-HEAD
  versions over newer evidence.
- Add Qortal skills that are genuinely reusable, but make them cite the platform
  pack and source revisions.
- Remove or merge skills that duplicate a platform guide without adding a
  reusable procedure.
- Add the skill quality gate and index coverage check.
- Document the precedence rule: platform source/runtime > platform pack >
  skill > workflow > historical report.
- Require a freshness result in every task.

**Exit criterion:** Skill index and validator pass; each skill has one platform,
one maturity, one source of truth, and no duplicate contract.

### Phase 4 — Unify task, handoff, and report flow

**Goal:** One task lifecycle across both platforms.

- Standardize `TASK.md`, `STATUS.json`, `REPORT.md`, and `HANDOFF.md`.
- Map Qortal report statuses to the shared state machine and document the
  mapping in one place.
- Decide the canonical report/evidence root and the disclosure rule.
- Require exact application SHA, branch, skill freshness, and executing agent.
- Add a single-writer rule for `STATUS` and idempotent resume rules.
- Add one validator that checks task artifacts, links, identity fields, and
  skill index consistency.

**Exit criterion:** A Qortal task and a Qortium task can both complete using the
same artifact shapes and the same validator.

### Phase 5 — Add thin client and service adapters

**Goal:** Make the environment usable by real clients without duplicating
knowledge.

- Add an OpenCode adapter that points to the canonical contract, workflow, and
  skills.
- Add a DeepSeek client boundary that resolves credentials externally and never
  writes them into the repository.
- Add read-only GitHub validation or a local pre-push check if authorized.
- Do not add autonomous dispatch, scheduled jobs, or background polling yet.
- Verify that every adapter can be removed without losing canonical knowledge.

**Exit criterion:** OpenCode/DeepSeek can follow the same contract; no adapter
contains a second policy or any secret.

### Phase 6 — Pilot one real task end to end

**Goal:** Prove the unified workflow rather than describe it.

- Select one small, bounded, non-destructive task.
- Start from the control plane, load only the matching platform pack and skills.
- Run the bounded implementation and validation.
- Produce the unified `TASK`/`STATUS`/`REPORT`/`HANDOFF` artifacts.
- Perform capability harvest and update the skill library if justified.
- Verify exact SHAs and remote evidence.
- Measure: time to context, duplicate research avoided, review rework, missing
  evidence, and human interventions.

**Exit criterion:** One complete, auditable task with truthful evidence and no
unresolved BLOCKER/HIGH finding.

### Phase 7 — Retire or archive duplicates with owner approval

**Goal:** Reduce maintenance without losing history.

- Mark the old platform repository as archived or read-only after the new pack
  is accepted.
- Keep historical reports available under the evidence policy.
- Redirect old absolute paths and document the move.
- Archive obsolete branches only after their unique content is inventoried and
  recoverable.
- Run a final validity check on links, provenance, skills, and task artifacts.

**Exit criterion:** One maintained control plane, one platform pack, one skill
library, one task schema, and a clear historical archive.

### Phase 8 — Introduce automation gradually

**Goal:** Add autonomy only where the workflow is already proven.

Priority order:

1. read-only validators on every change;
2. task artifact generation from templates;
3. freshness checks for skills and references;
4. optional GitHub issue/PR linking;
5. optional local dispatch through an authorized client;
6. only then consider scheduled or event-driven execution.

Every automation step requires explicit authorization, an audit trail, a
kill/stop path, and a secret boundary.

---

## 8. Key risks and mitigations

| Risk | Impact | Mitigation |
| --- | --- | --- |
| Consolidation accidentally overwrites owner work | Data loss | Freeze, back up, use new branches/worktrees, never reset or clean |
| Stale local HEAD treated as current | Wrong decisions | Record exact HEAD and remote refs; re-verify before migration |
| Old absolute paths break routing | Agents read wrong files | Replace with relative paths or one root variable; link validator |
| Two control planes persist | Governance drift | Owner decision in Phase 0; archive one after acceptance |
| Skills duplicate platform guides | Contradictory authority | Precedence rule; skills cite packs; deduplicate in Phase 3 |
| Template/status mismatch | Incomplete handoffs | One schema and one validator in Phase 4 |
| Secret leakage during DeepSeek/GitHub integration | Credential exposure | External secret store; no secrets in tasks/reports; pre-commit scan |
| Automation outruns governance | Unauthorized writes | No dispatch/write automation until Phase 8 and explicit authorization |
| Live node evidence mistaken for host proof | False completion | Preserve non-interchangeable validation layers |
| Qortal/Qortium contract leakage | Wrong platform behavior | Platform-scoped skills and packs; explicit boundary checks |
| Evidence volume overwhelms control repo | Maintenance burden | Separate evidence root or archive; retention policy |

---

## 9. Recommended immediate next actions

These are recommendations only; none was implemented.

1. Obtain owner approval for the canonical control plane and evidence location.
2. Back up both repositories and record exact branch/SHA inventories.
3. Reconcile the current VPS paths before any migration.
4. Decide whether to adopt the newest orchestration branch as the migration base
   instead of local `agent/orchestration/bootstrap`.
5. Preserve the Qortal standard and guides as a platform pack; do not rewrite
   them during the move.
6. Treat Shadow Archives reports as historical evidence, not active universal
   guidance.
7. Run the unified validator and one pilot task before archiving either
   repository.

---

## 10. Final recommendation

Consolidate toward **one control plane, two platform packs, one skill library, one
task/handoff schema, and thin client adapters**.

Use `ai-development-orchestration` as the architectural base because it already
has the correct separation of governance, roles, reusable capability, and
handoff state. Use `qortal-dev-workspace` as the first complete platform pack and
as the richest evidence source. Do not attempt a big-bang merge. Do not build an
autonomous scheduler yet. Prove one end-to-end task through the unified structure
first, then archive duplicates and add automation gradually.

Report saved:
`/home/iffi/dev-workspace/docs/workspace/audit/AGENT-WORKSPACE-COMPARISON.md`
