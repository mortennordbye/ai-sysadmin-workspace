# Inventory — Client-Alpha

The source of truth for the systems in scope for this client. Every `target-state.md`,
`approved-plan.md`, and handover for Client-Alpha should reference hosts by the identifiers
recorded here, so there is one authoritative answer to "what machine are we talking about?"

> **Do not record secrets here** (passwords, keys, tokens). Identifiers, roles, and ownership only.
> Keep this current — a plan built against a stale inventory is a plan built against the wrong box.

## Hosts

| Hostname / FQDN | Role | IP (mgmt) | OS / version | vSphere VM name | Owner / contact | Notes |
|-----------------|------|-----------|--------------|-----------------|-----------------|-------|
| _dc01.corp.example.local_ | _Legacy domain controller (all FSMO)_ | _10.0.0.10_ | _Windows Server (legacy)_ | _ALPHA-DC01_ | _[admin]_ | _Migration source_ |
| _dc02.corp.example.local_ | _New domain controller_ | _10.0.0.11_ | _Windows Server (target)_ | _ALPHA-DC02_ | _[admin]_ | _Migration target_ |
| _web01.corp.example.local_ | _RHEL web host_ | _10.0.1.20_ | _RHEL_ | _ALPHA-WEB01_ | _[admin]_ | _nginx target_ |

## Networks / Segments

| Segment | VLAN / port group | Range | Purpose |
|---------|-------------------|-------|---------|
| _Management_ | _[port group]_ | _10.0.0.0/24_ | _DC / infra management_ |

## Access Paths

| Path | How | Notes |
|------|-----|-------|
| _Jump host_ | _[bastion FQDN]_ | _Key-based; secrets in vault, not here_ |

## Change Log

| Date | Change | By |
|------|--------|----|
| _[YYYY-MM-DD]_ | _Inventory initialized_ | _[admin]_ |
