# Client-Alpha

Per-client working space. Everything specific to this engagement lives here so that work on one
client never bleeds into another. Copy this directory (rename it) to onboard a new client.

## Layout

```text
Client-Alpha/
├── README.md             # This file — engagement context
├── inventory.md          # Source of truth: the systems in scope for this client
├── active-handover.md    # Current context-reset checkpoint (see workspace README)
├── tasks/                # Live work for this client — one directory per task
│   └── <task-name>/
│       ├── target-state.md
│       ├── approved-plan.md
│       └── handover.md   # optional: per-task handover, archived when the task closes
└── secrets/              # GIT-IGNORED. Never commit credentials here — use a vault.
    └── README.md
```

## Engagement Context

Fill these in for the real client. Keep it factual; do not put credentials in this file.

| Field | Value |
|-------|-------|
| Client / environment name | Client-Alpha |
| Primary contact | [name, role, contact method] |
| Change-approval authority | [who signs off on `approved-plan.md`] |
| Maintenance windows | [agreed windows] |
| vSphere / management access path | [jump host / bastion, no secrets] |
| Notes / constraints | [anything an AI session must know before acting] |

## Working Here

1. Define each new unit of work as `tasks/<task-name>/target-state.md`.
2. Have the AI produce `tasks/<task-name>/approved-plan.md`; review and approve before executing.
3. Keep `inventory.md` current — it is the context every plan and handover relies on.
4. Add the task to the top-level `STATUS.md` so cross-client work stays visible.
5. On context bloat or an error loop, snapshot state into `active-handover.md` and start fresh.
