# Workspace Status — Open Work Across All Clients

A single, at-a-glance index of every task currently in flight, across all environments and
clients. The per-task `target-state.md` / `approved-plan.md` and the per-client handovers are the
detail; **this file is the radar**. Update it whenever a task starts, changes phase, or closes — so
nothing gets lost when you are juggling multiple environments.

> Convention: one row per active task. Move finished work to the **Recently Closed** section rather
> than deleting it, so the history of what changed (and when) is preserved.

## In Flight

| Client | Task | Path | Platform | Current phase / gate | Reversible? | Owner | Last update |
|--------|------|------|----------|----------------------|-------------|-------|-------------|
| _example_ | _AD migration_ | `customers/Client-Alpha/tasks/<task>/` | Windows Server | _Phase B — Gate G2 pending_ | _Yes — legacy DC still authoritative_ | _[admin]_ | _[YYYY-MM-DD]_ |

## Blocked / Needs Decision

| Client | Task | Blocker | Waiting on | Since |
|--------|------|---------|------------|-------|
| _(none)_ | | | | |

## Recently Closed

| Client | Task | Outcome | Closed |
|--------|------|---------|--------|
| _(none yet)_ | | | |
