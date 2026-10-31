# OpenCode tmux session persistence — workspace/implementation

Executing agent (registered role of the agent that actually produced this work): DeepSeek
Report/handoff writer (if different from the executing agent): DeepSeek
Executing-agent evidence (how the real executor was established): the runtime declares model ID `deepseek/deepseek-flash`; execution occurred in the OpenCode CLI harness. The harness is not the author.
Report type / category (audit, reports, research, implementation, validation, handoffs): implementation
Project ID / central report root: `orchestration` / `/home/iffi/dev-workspace/docs/workspace/`
Exact application repository / branch / SHA: control-plane `agent/control-plane/opencode-tmux-sessions-20261031` from `main` `982c33a9c7856f76bcecbae021ae5a084ff4da90`; docs `agent/workspace/opencode-tmux-session-report-20261031` from `main` `215deef5f66d3b48c9be8b843d82469bfb7e1b8b`.
Canonical report path `docs/workspace/implementation/2026-10-31-opencode-tmux-session-persistence.md` / SHA-256 recorded in the final response / remote evidence: control-plane PR https://github.com/iffinland/dev-control-plane/pull/4 and docs PR https://github.com/iffinland/dev-workspace-docs/pull/3.

## Objective and exit criterion

Implement and verify a simple tmux-based workflow that keeps OpenCode running
after a Termius/SSH disconnect and lets the owner reattach to the same terminal.
Document naming, working-directory, lifecycle, cleanup and operational limits.

Result: **PASS**. The tmux lifecycle, client disconnect survival, reattachment
and isolated cleanup were verified with a harmless heartbeat process. The
owner's OpenCode and Qortium processes were not affected. No service, linger,
scheduler or network listener was created.

## Verified baseline

| Item | Value |
| --- | --- |
| OS | Debian GNU/Linux 13 (trixie), kernel 6.12 |
| CPU / memory | 6 vCPU, 11 GiB RAM, no swap, about 8.1 GiB available |
| Disk | 197 GB root, about 142 GB free |
| OpenCode | `1.19.5`, `/home/iffi/.opencode/bin/opencode` |
| OpenCode session store | SQLite `~/.local/share/opencode/opencode.db` (about 39 MB); `opencode session list` showed persistent history |
| tmux | `3.5a`, `/usr/bin/tmux`; no user `.tmux.conf` |
| systemd | 257 user manager running; `Linger=no`; no user unit directory |
| Existing tmux/screen sessions | none |
| SSH | `ClientAliveInterval 120`; Termius is the client, not a server component |
| Git baseline | control-plane `main` `982c33a`; docs `main` `215deef`; both clean |

The current OpenCode process was a direct SSH TUI, not inside tmux (`TMUX`
unset). Its session history persists in SQLite, but terminal reattachment
requires a multiplexer such as tmux.

## Implementation

Added a control-plane operating runbook:
`control-plane/docs/operations/OPENCODE-TMUX-SESSIONS.md`.

It defines:

- session naming `oc-<project>-<task>-<YYYYMMDD>`;
- one repository/worktree per session, always started with an absolute `-c`
  working directory;
- start, detach, list, reattach, inspect, resume and safe-terminate commands;
- the boundary between terminal persistence and autonomous task execution;
- operational limits: one heavy OpenCode session, memory monitoring, no reboot
  persistence, no `opencode serve`/network listener, no systemd linger/service,
  and explicit owner authorization for unattended tasks.

Also linked the runbook from `README.md`, `WORKFLOW.md` and the OpenCode adapter.
No tmux configuration file was created because tmux defaults already provide the
required lifecycle. No package was installed.

## Test evidence

Harmless test session: `oc-selftest-20261031T073750Z-278528`, running a
heartbeat loop in `/tmp/opencode`.

| Step | Result |
| --- | --- |
| Baseline | No tmux server; OpenCode PID `270751`; Qortium Java PID `13615` |
| Create detached session | `tmux has-session` = yes; pane cwd `/tmp/opencode`; heartbeat advancing |
| Attach client | `tmux list-clients` = 1 |
| Client exits | Session remained; clients = 0; heartbeat advanced 3 to 7 lines |
| Reattach | `tmux list-clients` = 1 |
| Second client exits | Session remained; clients = 0; heartbeat advanced 8 to 11 lines |
| Safe cleanup | Only the test session killed; session no longer exists; zero tmux sessions remain |
| Owner processes | OpenCode PID `270751` and Qortium PID `13615` unchanged and running |
| Resource impact | Memory stayed about 3.6 GiB used / 8.1 GiB available; test process negligible |

The heartbeat log timestamps continued across both client exits, proving the
session and its process were not tied to the attached terminal.

Actual TCP/SSH teardown was **not** simulated because it would risk the owner's
live connection. The tested client-exit path exercises the same server-side
persistence that protects against an SSH/Termius disconnect. A reconnect from a
separate SSH login is the remaining real-network step for the owner to confirm
on the next session.

## Preservation and non-actions

- No systemd linger, service, timer, scheduler or polling loop was configured.
- No `opencode serve` process or network listener was started.
- No tmux configuration file or package was installed.
- No application repository, Qortium node configuration or runtime was modified.
- No actual overnight development task was started.
- No merge was performed.

## Files changed

- `control-plane/docs/operations/OPENCODE-TMUX-SESSIONS.md` (new runbook).
- `control-plane/README.md` (structure and main-document link).
- `control-plane/WORKFLOW.md` (unattended-session pointer).
- `control-plane/adapters/opencode/README.md` (session-persistence pointer).
- This report in the central docs repository.

## Checks not executed and why

- Real SSH/Termius network disconnect and reconnect: would risk the owner's
  active session; client exit/reconnect was tested instead.
- Reboot persistence: intentionally not implemented; tmux does not survive a
  reboot and systemd linger/services were outside authorization.
- `opencode serve`/remote attach: explicitly prohibited for this task.
- No application build/test/lint: no application source changed.

## Adversarial self-audit

- Used a unique test session name and killed only that session; never
  `tmux kill-server`.
- Verified `tmux list-clients`, `tmux has-session`, pane metadata and a monotonic
  heartbeat rather than assuming persistence from a detached process alone.
- Compared the owner's OpenCode and Qortium PIDs before and after; both were
  unchanged.
- Confirmed no new network listener and no systemd unit/linger change.
- Kept the runbook within the existing overnight-authorization governance rather
  than presenting tmux as autonomy.
- Did not install packages or create unnecessary configuration.

## External actions (commit/push/tag/release/deploy/QDN write/transaction)

- Documentation commits and normal branch pushes only, on dedicated feature
  branches.
- Control-plane PR: https://github.com/iffinland/dev-control-plane/pull/4
- Docs PR: https://github.com/iffinland/dev-workspace-docs/pull/3
- No merge, tag, release, deploy, QDN write, signing, transaction or server
  configuration change.

## Remaining risks and follow-up

1. The owner should confirm one real Termius/SSH reconnect against a tmux session
   during the first authorized use.
2. tmux sessions do not survive reboot; reboot persistence remains a separate
   owner decision requiring a systemd user service and lingering.
3. No swap: one heavy OpenCode session at a time; monitor memory.
4. tmux buffers and OpenCode logs may contain sensitive output; keep the socket
   user-private and lock the terminal.
5. `opencode serve` remains unconfigured and must not be exposed without a
   separate security decision.

## Report saved

- Canonical absolute path: `/home/iffi/dev-workspace/docs/workspace/implementation/2026-10-31-opencode-tmux-session-persistence.md`
- Category: `implementation`
- SHA-256 (optional): recorded in the final response
