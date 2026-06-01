# Active Handover — Client-Alpha

**Purpose:** This document is a context-reset checkpoint. When an AI session accumulates a large,
noisy history — or starts looping on a single error — the chat context degrades and hallucination
risk rises. Rather than continue in a bloated window, the administrator captures the exact current
state here, then **starts a fresh, sharp session** and re-seeds it from this single file.

> This is a **blueprint**. Replace the bracketed placeholders with real values. Treat every field
> as required: a handover that omits the machine state or the rollback position is not safe to
> resume from. Keep it factual and verbose — the next session knows only what is written here.

---

## 1. Engagement Metadata

| Field | Value |
|-------|-------|
| Customer | Client-Alpha |
| Active task | `customers/<client>/tasks/<task-name>/` (link the target-state and approved-plan) |
| Approved plan reference | `customers/<client>/tasks/<task-name>/approved-plan.md` (revision / approval date) |
| Handover author | [administrator name] |
| Handover timestamp | [absolute date and time, with timezone] |
| Reason for handover | [context window bloated / repeated error loop / shift change] |
| Current phase | [e.g., "Phase B — FSMO transfer, Gate G2 not yet passed"] |

---

## 2. Captured Machine and Environment State

Record the **exact** observed state, not the intended state. Paste real command output where
possible — the next session must not have to guess.

- **Hosts in scope:**
  - `[hostname / FQDN]` — role: `[e.g., new domain controller]` — IP: `[x.x.x.x]` —
    OS/version: `[...]` — vSphere VM name: `[...]`
  - `[hostname / FQDN]` — role: `[e.g., legacy domain controller]` — IP: `[x.x.x.x]` — ...
- **Snapshots / backups taken:** `[vSphere snapshot names + timestamps; System State backup ID]`
- **Key state evidence (paste output):**
  ```
  [e.g., output of `netdom query fsmo`, `repadmin /replsummary`, `dcdiag` summary]
  [or, for the RHEL example: `systemctl is-active nginx`, `firewall-cmd --list-services`]
  ```
- **Credentials / access context:** `[which privileged account/role is in use — never paste secrets]`
- **Maintenance window status:** `[open until HH:MM / closed]`

---

## 3. Completed Steps (with verification evidence)

List only steps that are **done and verified**. Each must cite the gate that confirmed it.

| Step | Verification gate | Evidence (command + result) | Confirmed by |
|------|-------------------|-----------------------------|--------------|
| [e.g., Phase A — new DC promoted] | G1 | `dcdiag /v` all pass; `repadmin /showrepl` clean | [admin] |
| ... | ... | ... | ... |

---

## 4. In-Progress Step

The single step currently underway. Be precise about where execution stopped.

- **Step:** `[step number and description from approved-plan.md]`
- **Last action executed:** `[exact command run and its full output, including any error]`
- **Observed result vs. expected:** `[what happened, and how it differs from the plan's PASS]`
- **Verification gate not yet passed:** `[e.g., G2 — FSMO roles not all showing on new DC]`

---

## 5. Remaining Tasks

Ordered list of what is left, lifted from the approved plan so the next session does not re-derive it.

1. [ ] `[next step + its verification gate]`
2. [ ] `[subsequent step]`
3. [ ] `[...]`
4. [ ] Final gate `[e.g., G4]` and capture of completion evidence.

---

## 6. Active Blockers and Error Traces

The reason the previous session struggled — captured verbatim so the fresh session does not repeat
the loop.

- **Blocker:** `[description]`
- **Exact error / trace:**
  ```
  [paste the full error message, event ID, or stack — not a paraphrase]
  ```
- **Hypotheses already ruled out:** `[what was tried and did not work — prevents re-looping]`
- **Open hypotheses to test next:** `[ranked next diagnostic steps]`

---

## 7. Rollback Position

Critical safety field: where the environment currently sits relative to the rollback plan.

- **Is the change currently reversible without data loss?** `[yes / no — explain]`
- **Active rollback assets:** `[snapshot names, backups, FSMO inventory baseline, config exports]`
- **Point of no return reached?** `[e.g., "No — legacy DC still online and authoritative" / "Yes — old DC demoted"]`
- **If resuming fails, the immediate rollback action is:** `[the specific first step from the
  approved-plan.md Rollback section that applies to the current phase]`

---

## 8. How to Resume in a Fresh Session

1. Start a **new** AI session (clean context) for Client-Alpha.
2. Provide the AI with: the rule file (`CLAUDE.md` / `GEMINI.md` / `.cursorrules`), the active
   `tasks/<task-name>/target-state.md` and `approved-plan.md`, and **this handover document**.
3. Instruct the AI to confirm the captured machine state (Section 2) against the live environment
   **before** taking any action — state may have changed since this snapshot was written.
4. Resume at the in-progress step (Section 4) / next remaining task (Section 5), honoring all
   verification gates and the Plan-First rule.
5. Once resumed and stable, archive this handover (rename with its close timestamp) and, if work
   continues, open a fresh `active-handover.md` at the next checkpoint.

---

## 9. Handover Log

| Timestamp | Author | Note |
|-----------|--------|------|
| [absolute date/time] | [admin] | Initial handover created at [phase]. |
