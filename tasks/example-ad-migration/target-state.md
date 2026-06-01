# Target State: Active Directory Domain Services Migration to a Newer Windows Server Version

**Task ID:** example-ad-migration
**Platform:** Windows Server (VMware vSphere virtualized)
**Document type:** Declarative target state (the "what", not the "how")
**Status:** Example / template

---

## 1. Objective

Migrate the Active Directory Domain Services (AD DS) role for the `corp.example.local` domain
from the existing legacy domain controller to a newly provisioned domain controller running a
newer, supported version of Windows Server. On completion, the legacy domain controller must be
cleanly decommissioned (gracefully demoted, not powered off) and the domain must continue to
operate without any interruption to authentication, DNS, or Group Policy services.

This is a **side-by-side migration**: the new domain controller is introduced alongside the
existing one, all roles and services are transferred, and only then is the old controller removed.
At no point should the domain be left without at least one healthy, advertising domain controller.

---

## 2. Scope

**In scope:**
- Provisioning one new domain controller VM on the existing vSphere platform.
- Promoting it into the existing domain/forest.
- Transferring all five Flexible Single Master Operation (FSMO) roles.
- Migrating DNS (AD-integrated) and confirming SYSVOL/Group Policy replication.
- Gracefully demoting and decommissioning the legacy domain controller.

**Explicitly out of scope (defer to `ai-backlog.md` if discovered):**
- Raising the domain or forest functional level.
- Restructuring Organizational Units, Group Policy Objects, or the DNS namespace.
- Migrating any additional roles (DHCP, file shares, certificate services) that may co-reside on
  the legacy host. These require their own separate target-state documents.
- Hardening or cipher/protocol changes on either host.

---

## 3. Assumptions

These assumptions **must be confirmed before planning execution**. If any is false, stop and revise.

1. The domain currently has exactly one legacy domain controller holding all FSMO roles, or the
   FSMO role placement is known and documented.
2. The existing forest and domain functional levels are supported by the target Windows Server
   version (i.e., the new OS can be promoted into this forest without a forced functional-level raise).
3. There is sufficient vSphere capacity (compute, datastore, and an appropriate VLAN/port group)
   to run both domain controllers concurrently for the duration of the migration.
4. A current, **tested** System State backup of the legacy domain controller exists or can be taken.
5. A defined maintenance window exists, even though no outage is expected, to contain blast radius.

---

## 4. Pre-Requisite Conditions

The following conditions must all be **true and verified** before any change is executed. These
are validated in the pre-change verification phase of the approved plan.

| # | Pre-requisite | How it is confirmed |
|---|---------------|---------------------|
| P1 | Existing AD replication is healthy with no errors | `repadmin /replsummary` and `repadmin /showrepl` report no failures |
| P2 | Legacy DC overall health is good | `dcdiag /v` passes all critical tests |
| P3 | A verified, recent System State backup exists | Backup catalog confirms a successful, restorable System State backup |
| P4 | DNS is functioning and AD-integrated zones resolve | `Resolve-DnsName` for domain SRV records (`_ldap._tcp.dc._msdcs.corp.example.local`) succeeds |
| P5 | Time synchronization is healthy on the PDC Emulator | `w32tm /query /status` shows a valid, authoritative source |
| P6 | New VM is provisioned, patched, statically addressed, and time-synced | VM exists on vSphere, OS patched to current baseline, static IP set, points DNS at the existing DC |
| P7 | A documented FSMO role inventory exists | `netdom query fsmo` output captured as the rollback reference |

---

## 5. Success Criteria

The migration is considered complete and successful only when **every** criterion below is met and
independently verified by a human administrator:

1. **New DC is promoted and advertising.** `nltest /dsgetdc:corp.example.local` returns the new
   domain controller, and `dcdiag /v` on the new DC passes all tests.
2. **All five FSMO roles reside on the new DC.** `netdom query fsmo` lists the new DC for Schema
   Master, Domain Naming Master, RID Master, PDC Emulator, and Infrastructure Master.
3. **Replication is clean in both directions.** `repadmin /replsummary` shows zero failures and a
   recent successful replication between the new and legacy DCs.
4. **DNS, SYSVOL, and Group Policy replicate to the new DC.** AD-integrated DNS zones are present
   on the new DC, the SYSVOL share is published (`net share` shows `SYSVOL` and `NETLOGON`), and
   GPO content is consistent (`dfsrmig /getmigrationstate` reports "Eliminated"/healthy DFSR state).
5. **Clients authenticate against the new DC.** A test workstation successfully logs on and
   `nltest /sc_query:corp.example.local` from a member server confirms a secure channel.
6. **Legacy DC is gracefully demoted and removed.** The old host is demoted via the standard
   process (not forcibly), its computer/NTDS metadata is gone, and `repadmin /viewlist *` no longer
   references it. The DNS namespace no longer lists the demoted host as a name server.
7. **No regressions.** `dcdiag /v` and `repadmin /replsummary` against the surviving DC(s) are clean
   after decommissioning.

---

## 6. Definition of Done

- All success criteria in Section 5 are met and the verification output is captured as evidence.
- The legacy DC VM is powered off (not deleted) and retained for a defined cool-down period as an
  additional rollback safety net before final removal from vSphere.
- Any unrelated issues observed during the work are logged to `ai-backlog.md`, not fixed in flight.
- An `approved-plan.md` was reviewed and approved by a human before any step was executed.
