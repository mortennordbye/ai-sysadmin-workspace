# Approved Plan: Active Directory Domain Services Migration to a Newer Windows Server Version

**Task ID:** example-ad-migration
**Satisfies:** `target-state.md` in this directory
**Platform:** Windows Server (VMware vSphere virtualized)
**Approval status:** Example / template — replace with a real human sign-off before execution

> **Plan First.** This document must be reviewed and explicitly approved by a human administrator
> before a single command is executed. Nothing in this plan is to be run automatically. Each
> numbered step is paired with a verification gate, and Section 7 contains a mandatory rollback
> procedure for every phase.

---

## 1. Assumptions and Blast Radius

**Assumptions** (carried forward from `target-state.md`, Section 3 — confirm before starting):
- A single legacy domain controller currently holds all FSMO roles for `corp.example.local`.
- The forest/domain functional level already supports the target Windows Server version.
- vSphere has capacity to run both domain controllers concurrently.
- A tested System State backup of the legacy DC exists.

**Blast radius if this goes wrong:**
- Domain-wide. A failed FSMO transfer, broken replication, or premature demotion can disrupt
  authentication, DNS resolution, and Group Policy for **every** domain-joined machine.
- Mitigation: the old DC stays online and healthy until every success criterion is verified; no
  destructive action (demotion, metadata cleanup) occurs until the new DC is fully validated.

**Maintenance window:** Perform during the approved low-usage window even though no outage is
expected, so that any unexpected behavior is contained.

---

## 2. Pre-Change Verification (run and confirm BEFORE any change)

Run each check on the **legacy** domain controller unless noted. Record the output as the baseline
and rollback reference.

| Check | Command | Expected PASS result |
|-------|---------|----------------------|
| Replication summary | `repadmin /replsummary` | No failures; all deltas recent |
| Replication detail | `repadmin /showrepl` | No `LAST FAILURE` entries |
| DC health | `dcdiag /v` | All critical tests pass |
| FSMO inventory | `netdom query fsmo` | All five roles listed; **record this output** |
| DNS SRV records | `Resolve-DnsName _ldap._tcp.dc._msdcs.corp.example.local -Type SRV` | Returns the existing DC |
| Time service | `w32tm /query /status` | Valid, authoritative source on the PDC Emulator |
| System State backup | Backup console / `wbadmin get versions` | A recent, successful, restorable backup exists |

> **STOP — Verification Gate G0.**
> Do not proceed unless every pre-change check above passes. If replication or `dcdiag` reports
> errors, the environment is **not** healthy enough to migrate. Stop, report the specific failure,
> and remediate (or escalate) before continuing. Do not attempt to "push through" a dirty baseline.

---

## 3. Phase A — Provision and Promote the New Domain Controller

1. Confirm the new VM meets requirements: correct Windows Server version, fully patched, static IP,
   and DNS client pointed at the **existing** legacy DC.
2. Take a vSphere snapshot of the **new** VM only (pre-promotion), so the promotion itself is
   reversible without touching the live DC.
3. Prepare the forest and domain schema (only required when the target OS introduces schema
   changes for this forest). From an account in Schema Admins / Enterprise Admins, using the target
   OS installation media:
   ```
   adprep /forestprep
   adprep /domainprep
   ```
4. Install the AD DS role and promote the new server as an **additional** domain controller in the
   existing domain (replica promotion, *not* a new forest):
   ```powershell
   Install-WindowsFeature AD-Domain-Services -IncludeManagementTools
   Install-ADDSDomainController `
     -DomainName "corp.example.local" `
     -InstallDns:$true `
     -Credential (Get-Credential) `
     -NoGlobalCatalog:$false
   ```
5. Allow the server to reboot and complete promotion.

> **STOP — Verification Gate G1.**
> On the new DC, run `dcdiag /v` (all tests pass) and `repadmin /showrepl` (inbound replication
> from the legacy DC succeeded). Confirm `net share` lists `SYSVOL` and `NETLOGON`. Confirm the new
> DC is a Global Catalog (`Get-ADDomainController -Identity <newDC> | Select IsGlobalCatalog`).
> Do not proceed to FSMO transfer until replication is clean in both directions.

---

## 4. Phase B — Transfer FSMO Roles

1. Allow at least one full successful replication cycle after G1 before moving roles.
2. Transfer (do **not** seize) all five FSMO roles to the new DC. Run from the new DC:
   ```powershell
   Move-ADDirectoryServerOperationMasterRole `
     -Identity "NEW-DC-NAME" `
     -OperationMasterRole SchemaMaster,DomainNamingMaster,RIDMaster,PDCEmulator,InfrastructureMaster
   ```
   Confirm the transfer when prompted. `Move-...` performs a graceful transfer; reserve a seizure
   for the rollback/failure path only.

> **STOP — Verification Gate G2.**
> Run `netdom query fsmo`. All five roles must now report the new DC. Re-run `repadmin /replsummary`
> and confirm zero failures. If any role failed to transfer, **do not seize** as a first reaction —
> investigate connectivity/replication first (see Rollback, Section 7, Phase B).

---

## 5. Phase C — Migrate DNS Resolution and Repoint Clients

1. Confirm AD-integrated DNS zones are present and resolving on the new DC:
   ```powershell
   Get-DnsServerZone
   Resolve-DnsName corp.example.local -Server NEW-DC-IP
   ```
2. Update DHCP scope options and any statically configured servers so that DNS resolution now
   includes the new DC. **Retain the legacy DC as a secondary DNS entry** during the transition;
   do not remove it from client configuration yet.
3. Validate from a member server / test workstation:
   ```
   nltest /dsgetdc:corp.example.local
   nltest /sc_query:corp.example.local
   ```

> **STOP — Verification Gate G3.**
> A test workstation must successfully authenticate and resolve domain resources using the new DC.
> Group Policy must apply (`gpupdate /force` succeeds; `gpresult /r` shows policies sourced
> correctly). Do not proceed to decommissioning until clients are confirmed healthy.

---

## 6. Phase D — Gracefully Demote and Decommission the Legacy DC

> Execute this phase **only** after G0–G3 have all passed and the new DC has run as the sole
> FSMO holder, healthy, through at least one business cycle (per the approved window).

1. On the legacy DC, demote it gracefully (this removes AD DS and updates metadata automatically):
   ```powershell
   Uninstall-ADDSDomainController -DemoteOperationMasterRole:$false -Credential (Get-Credential)
   ```
2. After demotion and reboot, from the new DC confirm the old DC is gone from the directory:
   ```powershell
   Get-ADDomainController -Filter *
   repadmin /viewlist *
   ```
3. Remove the legacy DC's lingering DNS name-server (NS) records and host records from the
   AD-integrated zones if they were not cleaned automatically.
4. Remove the legacy DC from client/DHCP DNS configuration now that it is decommissioned.
5. **Power off** (do not delete) the legacy DC VM and retain it for the defined cool-down period.

> **STOP — Verification Gate G4 (final).**
> On the surviving DC, `dcdiag /v` and `repadmin /replsummary` must be clean, `netdom query fsmo`
> must list the new DC for all roles, and no references to the old DC may remain
> (`repadmin /viewlist *`). Capture all output as completion evidence. Only then is the task done.

---

## 7. Rollback Plan (Mandatory)

Rollback strategy is **phase-specific**: the safest reversal depends on how far the migration
progressed. The guiding principle is that the legacy DC remains the authoritative fallback until
the very end, so most failures are recovered by simply not proceeding and reverting the new DC.

### Phase A failure (promotion of the new DC failed or replication is dirty)
- The legacy DC is untouched and remains fully authoritative; the domain is unaffected.
- If AD DS partially installed on the new DC, demote it: `Uninstall-ADDSDomainController`.
- If demotion is not possible, revert the new VM to the pre-promotion vSphere snapshot taken in
  Phase A step 2, then run metadata cleanup on the legacy DC to remove the failed DC object:
  ```powershell
  ntdsutil "metadata cleanup" "remove selected server NEW-DC-NAME" quit quit
  ```
- Re-run `repadmin /replsummary` and `dcdiag /v` on the legacy DC to confirm health restored.

### Phase B failure (FSMO transfer failed or roles split between DCs)
- **Do not panic-seize.** First confirm connectivity and replication between the two DCs.
- If the transfer partially completed and the new DC is healthy, complete the remaining transfers
  once replication recovers.
- If the new DC is unhealthy/unreachable and roles are stranded, **seize** the roles back to the
  legacy DC (seizure is the emergency-only path), then demote/clean up the new DC as in Phase A:
  ```powershell
  Move-ADDirectoryServerOperationMasterRole -Identity "LEGACY-DC-NAME" `
    -OperationMasterRole SchemaMaster,DomainNamingMaster,RIDMaster,PDCEmulator,InfrastructureMaster -Force
  ```
  After any seizure, the DC whose roles were seized must **never** be brought back online without a
  full metadata cleanup and rebuild — power it off and treat it as failed.

### Phase C failure (clients cannot authenticate or resolve via the new DC)
- Revert DNS/DHCP client configuration to point primarily at the legacy DC (which is still online).
- Leave the new DC in place but investigate DNS zone replication and secure-channel issues before
  re-attempting the repoint. No destructive action required; the legacy DC continues serving.

### Phase D failure (problems surface after the legacy DC was demoted)
- This is the highest-risk phase, which is why the legacy DC VM is **powered off, not deleted**.
- If the new DC fails after decommissioning and no other healthy DC exists, restore the legacy DC
  from the verified System State backup (P3) to a recovery host, or, if within the cool-down window
  and the legacy VM was only demoted-then-powered-off, perform an authoritative/non-authoritative
  restore as appropriate. Engage AD recovery procedures and escalate before acting.
- Re-establish a single authoritative DC first, verify with `dcdiag`/`repadmin`, then re-plan.

### Universal rollback safeguards
- vSphere snapshot of the new VM taken pre-promotion (Phase A).
- Verified System State backup of the legacy DC (pre-requisite P3).
- Legacy DC VM retained, powered off, through the cool-down period after Phase D.
- FSMO inventory (`netdom query fsmo`) captured at G0 as the authoritative "known good" reference.

---

## 8. Post-Task Actions

- Capture all verification-gate output (G0–G4) as completion evidence for the change record.
- Log any unrelated misconfigurations discovered during the work to `ai-backlog.md` — do not fix
  them as part of this task.
- After the cool-down period and a final stakeholder confirmation, remove the retained legacy DC VM
  from vSphere as a separate, explicitly approved cleanup task.
