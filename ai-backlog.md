# AI Backlog — Deferred Technical Debt and Side Findings

This file is the single, append-only register for issues discovered **incidentally** while working
on an approved task. Per the workspace rules (the Backlog Rule in `CLAUDE.md` / `GEMINI.md` and
`scope_management.backlog_enforcement` in `.cursorrules`):

- **Do not fix items found here as part of an unrelated task.** Log them and keep going.
- Each entry must be self-contained: enough context that someone can later scope it into its own
  task with its own target state, plan, and rollback.
- Nothing leaves this backlog except by being promoted into an explicitly approved task.

## How to Add an Entry

1. Assign the next sequential ID (`BL-NNN`).
2. Record the date discovered (absolute date, not "today").
3. Note the host/scope, severity, what you observed, and — most importantly — **why it was deferred**.
4. Do not act on it. Mention it to the user and move on.

**Severity guide:** `Low` (cosmetic / hygiene) · `Medium` (should be fixed in a planned window) ·
`High` (security or stability risk; schedule promptly, but still do not fix mid-task).

---

## Open Items

### BL-001 — Non-standard / weak SSH ciphers on RHEL fleet
- **Date discovered:** 2026-06-01
- **Host / scope:** Observed on the RHEL target during the `example-nginx-ansible` connectivity
  check; likely fleet-wide given a shared base image.
- **Severity:** High (security hardening)
- **Description:** The SSH daemon advertises legacy cipher and MAC algorithms that fall outside the
  organization's intended hardening baseline. No standardized `Ciphers`/`MACs`/`KexAlgorithms`
  policy is currently enforced across the RHEL hosts.
- **Why deferred:** Tightening SSH algorithms is a fleet-wide change with real lockout risk
  (especially for older management tooling/jump hosts). It is unrelated to installing nginx and
  must be its own task with a tested rollback and a defined cipher standard. **Do not change SSH
  configuration mid-migration.**
- **Suggested next step:** Open a dedicated task to define and roll out a hardened, tested
  `sshd_config` cipher/MAC/KEX baseline via Ansible, with a `--check` dry run and per-host fallback.

### BL-002 — Untracked / undocumented registry key on Windows host
- **Date discovered:** 2026-06-01
- **Host / scope:** Legacy domain controller, observed during the `example-ad-migration` pre-checks.
- **Severity:** Medium
- **Description:** A non-default registry value was found under a service hive that does not match
  any documented baseline or change record. Its origin and purpose are unknown; it may be a stale
  manual tweak from a prior administrator.
- **Why deferred:** Modifying or deleting an undocumented registry key during an AD migration is
  reckless — its removal could change service behavior at the worst possible time, and it is wholly
  unrelated to the migration objective. Investigation must happen on a non-critical window.
- **Suggested next step:** Open a task to research the key's purpose (change history, vendor docs,
  test VM reproduction), then decide on documentation vs. removal with a registry export as rollback.

### BL-003 — Orphaned DNS records for a decommissioned host
- **Date discovered:** 2026-06-01
- **Host / scope:** AD-integrated DNS zone `corp.example.local`.
- **Severity:** Low
- **Description:** Stale A and PTR records reference a host that no longer appears to exist on the
  network. They were noticed while validating DNS during the AD migration pre-checks.
- **Why deferred:** DNS record cleanup is unrelated to the active migration and carries a small but
  real risk of removing a record that is still referenced somewhere. It belongs in a routine DNS
  hygiene task, not in the migration.
- **Suggested next step:** Open a DNS hygiene task to confirm the host is truly retired (monitoring,
  ARP, ticket history) before removing the records, exporting the zone first as rollback.

### BL-004 — Stale firewalld rule of unknown origin
- **Date discovered:** 2026-06-01
- **Host / scope:** RHEL target host, noted while reviewing `firewall-cmd --list-services`.
- **Severity:** Medium
- **Description:** A permitted service/port is present in the active firewalld zone that does not
  correspond to any currently running service or documented requirement.
- **Why deferred:** Closing an unexplained firewall opening could break an undocumented integration.
  It is unrelated to the nginx task and must be validated before removal.
- **Suggested next step:** Open a task to trace the rule's purpose (owning service, traffic logs,
  change records), then close it in a controlled window with the prior ruleset captured as rollback.

---

## Promoted / Closed Items

_(None yet. When an item is promoted into its own approved task, move it here with a link to the
task directory and the date promoted. Do not delete history.)_
