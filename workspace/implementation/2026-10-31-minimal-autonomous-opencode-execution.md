# Minimal autonomous OpenCode execution — workspace/implementation

Executing agent (registered role of the agent that actually produced this work): DeepSeek
Report/handoff writer (if different from the executing agent): DeepSeek
Executing-agent evidence (how the real executor was established): The running OpenCode session declares model `deepseek/deepseek-flash` under OpenCode `1.19.5`; execution occurred through the OpenCode CLI harness. The harness is not the author. Session ID: `ses_e6f0ae47cffeuVMdhk6XNpkXQa`.
Report type / category (audit, reports, research, implementation, validation, handoffs): implementation
Project ID / central report root: `orchestration` / `/home/iffi/dev-workspace/docs/workspace/`
Exact application repository / branch / SHA: control-plane `agent/control-plane/unattended-opencode-20261031` at `493125df09845c5fe57d5c817695548d99075c89`, based on `main` `97a843a0bb054accc2f7a119876704faa66fe73a`. Docs `agent/workspace/unattended-opencode-20261031` based on `main` `ef782abde8913fc06552b27ec510d43940414488`; the final docs commit is recorded in the handoff.
Canonical report path `docs/workspace/implementation/2026-10-31-minimal-autonomous-opencode-execution.md` / SHA-256 recorded in the final response / remote evidence: control-plane PR https://github.com/iffinland/dev-control-plane/pull/5 and the docs PR recorded in the follow-up commit.

## Verdict

**PASS**, with documented limitations.

The minimal unattended execution path is implemented: a separate restricted agent
profile, a no-daemon guard with preflight/thresholds/one-execution lock/four-hour
timeout/safe process-group cancellation, backward-compatible task-state fields,
a runbook, automated tests, a harmless dry run, validators and a passing CI run.
No scheduler, service, timer, listener, swap, systemd or application-repository
change was made.

Limitations are explicit: a live `opencode run` timeout was not executed because
it would create a second heavyweight OpenCode execution while the owner's
interactive session is active; the guard's OpenCode-specific timeout path was
tested with disposable process groups instead. A timeout is not a graceful
checkpoint; OpenCode session history is preserved for manual resumption.

## Objective and exit criterion

Implement the owner-approved minimal autonomous OpenCode execution design from
the 2026-10-31 capability audit, without building a second orchestration system.
Exit criterion: working restricted profile, guard, timeout/cancellation, resource
preflight, Qortium safety, session-history preservation, tests, dry run,
validators, committed branches, PRs and this report.

Achieved.

## Baseline

| Check | Result |
| --- | --- |
| control-plane PR #4 | merged; `main` `97a843a` contains `013a7e6` |
| docs PR #3 | merged; `main` `ef782ab` contains `616149c` |
| Branches | control-plane and docs feature branches created from synced `main` |
| Working trees before work | clean in control-plane; docs had the prior audit report untracked and preserved |
| Hooks | `core.hooksPath=.githooks` in both repositories |
| Validators before work | control-plane, skills, security, security self-test and Git-hook tests passed |
| Existing OpenCode session | preserved; PID `283341` was not terminated |
| Qortium | PID `13615`; preserved and never signalled |

The previous audit report remains at
`/home/iffi/dev-workspace/docs/workspace/audit/2026-10-31-autonomous-execution-capability-gap-audit.md`
with SHA-256 `cc225d39dcbd85c6590bdbe9889bcefc455243df7f7382ab7282c4cc29ab1651`.
It is not part of this implementation commit.

## Implemented components

| Component | Path | Status |
| --- | --- | --- |
| Restricted agent profile | `control-plane/adapters/opencode/agents/opencode-unattended.md` | Added |
| Installed agent profile | `~/.config/opencode/agents/opencode-unattended.md` | Added; byte-identical to canonical profile |
| Execution guard | `control-plane/tools/opencode_guard.py` | Added |
| Guard tests | `control-plane/tools/test_opencode_guard.py` | Added; 27 tests |
| Unattended runbook | `control-plane/docs/operations/UNATTENDED-OPENCODE.md` | Added |
| tmux runbook update | `control-plane/docs/operations/OPENCODE-TMUX-SESSIONS.md` | Owner-reported real Termius reconnect recorded; link to unattended runbook |
| OpenCode adapter | `control-plane/adapters/opencode/README.md` | Capability/limit notes and links |
| Workflow pointer | `control-plane/WORKFLOW.md` | Unattended mechanics pointer |
| README pointer | `control-plane/README.md` | Runbook link |
| TASK template | `control-plane/templates/TASK.md` | Optional unattended fields added; existing fields preserved |
| STATUS template | `control-plane/templates/STATUS.json` | Optional additive `unattended` object; `schema_version` remains `2` |
| Required-file validator | `control-plane/tools/validate_control_plane.py` | New runbook/profile/guard required |
| CI | `control-plane/.github/workflows/validate.yml` | Runs the guard tests |

### Agent profile

The profile is `mode: primary`, `steps: 80`, with `task: deny`, `question: deny`,
`plan_enter`/`plan_exit: deny`, and explicit allow/deny rules. It denies
`--auto`-style broad approval by design (the runbook forbids `--auto`), denies
secrets/key edits, and denies high-risk Bash classes including `sudo`,
`systemctl`, `systemd-run`, reboot/shutdown, `kill`/`pkill`/`killall`,
`tmux kill-server`, `opencode serve`/`web`/`acp`, `git push`, `git merge`, `gh`,
`ssh`/`scp`/`rsync`, container/orchestration commands, package publication and
`qortium*`/`qortal*`.

`opencode debug agent opencode-unattended` was verified locally: mode `primary`,
steps `80`, `question` deny, `task` deny, `git push *` deny, `qortium*` deny and
`/home/iffi/qortium-preview/**` external-directory deny.

The existing interactive configuration was preserved. The global
`~/.config/opencode/opencode.jsonc` SHA-256 before and after was
`88c857f59048f4d2165a673cab56c32541801afb58d30fe4df70d53384f29b45`.
The new profile is an additional file in the global `agents/` directory, not a
change to the interactive `build` agent or global permission policy.

### Guard

`opencode_guard.py check` reads memory, load, disk, direct heavyweight OpenCode
processes and the Qortium PID/RSS. `opencode_guard.py run` adds a lock, preflight,
logging, a four-hour maximum and safe cancellation.

| Behaviour | Implementation |
| --- | --- |
| RAM warning / refusal | `<3 GiB` exit `10`; `<1.5 GiB` exit `20` |
| Load warning | five-minute load `>5` exit `10` |
| Disk warning | available disk `<20 GiB` exit `10` |
| One heavyweight execution | a second direct OpenCode execution is refused (`20`); a lock prevents concurrent guard runs |
| Wall-clock maximum | `4h` maximum; lower values allowed; timeout exit `124` |
| Cancellation | `SIGINT` -> `130`; `SIGTERM` -> `143`; child process group receives `SIGTERM`, grace period, then `SIGKILL` |
| Qortium safety | Qortium is detected read-only; the guard only signals the process group it created and rejects unsafe targets |
| Session history | never deleted; timeout/signal logs the recent session list and resume hint |
| Exit 0 | explicitly logged as not acceptance |

## Tests and verification

All commands were run from `/home/iffi/dev-workspace/control-plane` unless noted.

| Check | Result |
| --- | --- |
| `python3 tools/validate_control_plane.py` | PASS — 46 active Markdown files, 136 links, 23 required files |
| `python3 tools/validate_skills.py` | PASS — 16 skills |
| `python3 tools/validate_security.py .` | PASS — 97 files scanned |
| `python3 tools/validate_security.py --self-test` | PASS |
| `python3 tools/test_git_hooks.py` | PASS |
| `python3 tools/test_opencode_guard.py` | PASS — 27 tests in about 21s |
| `python3 tools/opencode_guard.py check` | PASS at the audit baseline |
| control-plane CI run `41184617033` | completed/success |

The 27 guard tests cover duration parsing and the four-hour cap, RAM/load/disk
thresholds, one-session detection, helper-command exclusion, Qortium detection,
unsafe-target rejection, guard lock conflict, timeout termination of a child and
grandchild, SIGINT cancellation, unrelated-process-group survival, permission
profile restrictions and installed-profile loading.

### Harmless dry run

The owner authorized a harmless local dry run. It did not start an OpenCode model
session and did not run application development.

Normal guard run:

```text
python3 control-plane/tools/opencode_guard.py run \
  --timeout 5s \
  --log /home/iffi/dev-workspace/logs/opencode-dryrun-20261031T082458Z.log \
  -- sh -c 'echo dry-run-start; sleep 1; echo dry-run-end'
```

Result: exit `0`, both dry-run lines present, log records the pre/post resource
snapshot, Qortium PID `13615` unchanged.

Timeout guard run:

```text
python3 control-plane/tools/opencode_guard.py run \
  --timeout 1s --term-grace 1 \
  --log /home/iffi/dev-workspace/logs/opencode-dryrun-timeout-20261031T082458Z.log \
  -- sh -c 'echo timeout-start; sleep 30; echo timeout-end'
```

Result: exit `124`; log records `process group ... still alive; sending SIGKILL`
and the timeout recovery note; exact `sleep 30` process check found no orphan;
Qortium PID `13615` unchanged. The guard's SIGINT cancellation path is covered by
the automated test, which verifies the child and grandchild exit and the guard
returns `130`.

The live `opencode run` timeout path was not executed because the owner's
interactive OpenCode session is a heavyweight execution and the guard correctly
refuses a second one. OpenCode session history was verified to remain available
(`opencode session list --format json` still lists the existing sessions).

### Resource evidence

Observed during the dry run:

- RAM available: about 7.4 GiB of 11.7 GiB; swap 0;
- load: about 1.0-1.5;
- disk: about 141 GiB available;
- heavyweight OpenCode: one (PID `283341`);
- Qortium: PID `13615`, RSS about 3.1 GiB, read-only;
- no tmux server, no OpenCode listener, no systemd unit/timer/linger change;
- no application repository or Qortium configuration/runtime change.

## Exact operation commands

Start an authorized four-hour task from a shell/tmux session. The restricted
profile deliberately denies feature-branch pushes and PR creation; the owner
integrates local work after review unless a separately approved profile is used.

```bash
SESSION=oc-<project>-<task>-<YYYYMMDD>
WORKDIR=/home/iffi/dev-workspace/projects/<repository>
tmux new-session -s "$SESSION" -c "$WORKDIR"
```

Inside the tmux session:

```bash
python3 /home/iffi/dev-workspace/control-plane/tools/opencode_guard.py run \
  --timeout 4h \
  --log "/home/iffi/dev-workspace/logs/opencode-<task>-$(date -u +%Y%m%dT%H%M%SZ).log" \
  -- opencode run \
    --agent opencode-unattended \
    --model deepseek/deepseek-flash \
    --title "<task title>" \
    "<authorized task prompt>"
```

The guard refuses if another heavyweight OpenCode execution is active. Exit the
interactive TUI normally or wait; do not kill the owner's session.

Reconnect and inspect:

```bash
tmux attach-session -t "$SESSION"
tmux capture-pane -p -t "$SESSION" -S -100
tail -n 100 /home/iffi/dev-workspace/logs/opencode-<task>-<timestamp>.log
opencode session list --format json --max-count 10
opencode export <session-id> --sanitize
opencode --session <session-id>
```

Resume non-interactively after interruption:

```bash
opencode run --session <session-id> --agent opencode-unattended "<next instruction>"
```

## Timeout and recovery

A timeout returns `124`; SIGINT returns `130`; SIGTERM returns `143`. The guard
terminates the process group it created, not Qortium and not unrelated processes.
A child that deliberately creates a new session can escape the group; this is a
documented limitation.

A timeout is not a graceful checkpoint. The guard logs the timeout and recent
session list, but the agent may not have written its final checkpoint. The
OpenCode SQLite session history is preserved; resume with `--session <id>`, read
the log and task/STATUS artifacts, and verify the worktree before continuing.
Exit `0` is not acceptance; the task report, exact diff, checks and runtime
evidence are required.

## Git and CI

- control-plane branch: `agent/control-plane/unattended-opencode-20261031`
- control-plane commit: `493125df09845c5fe57d5c817695548d99075c89`
- control-plane PR: https://github.com/iffinland/dev-control-plane/pull/5
- control-plane CI: run `41184617033`, completed/success
- docs branch: `agent/workspace/unattended-opencode-20261031`
- docs PR: created from this branch after the report commit; the exact URL is
  recorded in the follow-up commit and final handoff

No merge was performed. The docs repository has no CI workflow.

## Files changed

Control-plane:

- `.github/workflows/validate.yml`
- `README.md`
- `WORKFLOW.md`
- `adapters/opencode/README.md`
- `adapters/opencode/agents/opencode-unattended.md`
- `docs/operations/OPENCODE-TMUX-SESSIONS.md`
- `docs/operations/UNATTENDED-OPENCODE.md`
- `templates/STATUS.json`
- `templates/TASK.md`
- `tools/validate_control_plane.py`
- `tools/opencode_guard.py`
- `tools/test_opencode_guard.py`

Owner environment:

- `~/.config/opencode/agents/opencode-unattended.md` (new; existing
  `opencode.jsonc` unchanged)

Docs:

- this report.

No application repository, Qortium configuration/runtime, systemd unit, swap
configuration, service, timer, scheduler or network listener was changed.

## Checks not executed and why

- A live `opencode run` timeout: would create a second heavyweight OpenCode
  execution while the owner's interactive session is active; the guard would
  refuse it. The timeout/cancellation mechanics were tested with disposable
  process groups instead.
- A real application-development task: prohibited by the task; the dry run was
  harmless.
- A graceful checkpoint at timeout: not guaranteed by OpenCode or the guard; the
  limitation is documented and session history is preserved.
- A real Termius disconnect during this task: the owner reported a successful
  reconnect on 2026-10-31; that is recorded as owner-provided evidence in the
  tmux runbook, not re-simulated here.
- A hard monetary cap: no such OpenCode 1.19.5 feature exists; no cost monitor
  was added, per the owner decision.
- A reboot-persistence service: deliberately not added.

## Adversarial self-audit

- Did not terminate, migrate or reconfigure the owner's OpenCode session.
- Did not signal, restart or modify Qortium; PID `13615` was unchanged.
- Did not use `--auto`, `pkill`, `killall`, `tmux kill-server` or a global
  process-name kill in the guard.
- Did not create a service, timer, scheduler, daemon, listener, swap or systemd
  configuration.
- Did not modify the existing global OpenCode permission policy; the global
  config SHA-256 was unchanged.
- Used a separate restricted profile and installed it as an additive file.
- Made STATUS changes additive and kept `schema_version` at `2`.
- Tested timeout/orphan cleanup, SIGINT cancellation, lock conflict, one-session
  refusal, resource thresholds and Qortium-safe targeting.
- Documented that permission patterns are not a sandbox and that a child can
  escape the process group by creating a new session.
- Did not claim graceful completion at timeout or acceptance from exit code `0`.
- Did not commit or push application repositories.

## External actions

- Committed and pushed the control-plane feature branch and opened PR #5.
- Created and installed the restricted agent profile.
- Ran a harmless local dry run and wrote its logs under
  `/home/iffi/dev-workspace/logs/`.
- No merge, deployment, QDN publication, signing, transaction, application
  repository change or Qortium change.

## Remaining risks and follow-up

1. The restricted profile denies feature-branch pushes and PR creation; the owner
   must integrate local work or approve a separate external-write profile.
2. No live `opencode run` timeout was exercised in this environment; the first
   authorized real task should confirm the timeout and resume path.
3. No graceful checkpoint is guaranteed at timeout; rely on session history,
   logs and STATUS artifacts.
4. The guard's one-session check recognises direct OpenCode invocations, not an
   OpenCode process hidden inside an arbitrary wrapper.
5. Permission patterns are not a security sandbox.
6. A child that creates a new session can escape process-group cancellation.
7. tmux does not survive a reboot; no reboot persistence was added.
8. No hard monetary cap; the four-hour maximum and 80-step limit are proxies.
9. OpenCode upgrades may change flags, permission behaviour or output events;
   re-run the capability checks after an upgrade.

## Capability harvest

The reusable operational procedure is now canonical in
`control-plane/docs/operations/UNATTENDED-OPENCODE.md`. A separate shared skill was
not created because the live `opencode run` timeout path was not exercised and
the procedure remains control-plane operational guidance. Revisit a candidate
`skills/shared/opencode-unattended-sessions/SKILL.md` after the first authorized
real task and owner-confirmed resume.

## Report saved

- Canonical absolute path: `/home/iffi/dev-workspace/docs/workspace/implementation/2026-10-31-minimal-autonomous-opencode-execution.md`
- Category: `implementation`
- SHA-256: computed after the final write and recorded in the final response
