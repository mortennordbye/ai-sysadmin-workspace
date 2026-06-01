# Approved Plan: Install nginx on RHEL via an Ansible Playbook

**Task ID:** example-nginx-ansible
**Satisfies:** `target-state.md` in this directory
**Platform:** Red Hat Enterprise Linux (VMware vSphere virtualized)
**Approval status:** Example / template — replace with a real human sign-off before execution

> **Plan First.** This plan must be reviewed and explicitly approved by a human before execution.
> Crucially, the change is validated with `ansible-playbook --check` (dry run) **before** it is
> applied for real. Section 6 contains a mandatory rollback procedure.

---

## 1. Assumptions and Blast Radius

**Assumptions** (from `target-state.md`, Section 3 — confirm before starting):
- Supported RHEL target with subscription/repo access providing `nginx`.
- Control node has key-based SSH and `become` privilege to the target.
- `firewalld` is active; port 80/tcp is currently free.

**Blast radius:**
- Single-host and small. The change installs one package, starts one service, and opens one
  firewall service. The main risks are (a) opening port 80 more broadly than intended and (b) a
  port conflict if something already listens on 80. Both are caught by the pre-checks below.

---

## 2. Pre-Change Verification (run and confirm BEFORE any change)

Run from the Ansible control node. Substitute the real inventory host/group for `<host>`.

| Check | Command | Expected PASS result |
|-------|---------|----------------------|
| Ansible present | `ansible --version` | Supported version returned |
| Connectivity | `ansible <host> -m ping` | `SUCCESS` / `pong` |
| Privilege escalation | `ansible <host> -b -m command -a 'id'` | Returns `uid=0(root)` context |
| Package available | `ansible <host> -b -m command -a 'dnf info nginx'` | Shows an available `nginx` package |
| Port 80 free | `ansible <host> -b -m shell -a 'ss -tlnp \| grep ":80 " \|\| echo FREE'` | Returns `FREE` |

> **STOP — Verification Gate G0.**
> Do not proceed unless all pre-checks pass. If port 80 is already bound, **stop** and report which
> service holds it — do not kill or reconfigure that service as part of this task. If `dnf info`
> cannot find `nginx`, stop and resolve subscription/repository access first.

---

## 3. The Playbook

Keep the playbook minimal — the rules favor the smallest configuration that solves the problem.
Save as `install-nginx.yml`.

```yaml
---
- name: Install and enable nginx on RHEL
  hosts: "{{ target_hosts | default('web') }}"
  become: true
  tasks:
    - name: Ensure nginx package is installed
      ansible.builtin.dnf:
        name: nginx
        state: present

    - name: Ensure nginx service is started and enabled
      ansible.builtin.service:
        name: nginx
        state: started
        enabled: true

    - name: Ensure HTTP is permitted through firewalld
      ansible.posix.firewalld:
        service: http
        permanent: true
        immediate: true
        state: enabled
```

Notes:
- `state: present` (not `latest`) keeps the run deterministic and idempotent.
- `permanent: true` plus `immediate: true` applies the firewall rule now **and** persists it across
  reloads/reboots, satisfying success criterion 4.

---

## 4. Execution

1. **Dry run first (mandatory).** This shows exactly what would change without changing anything:
   ```
   ansible-playbook -i inventory install-nginx.yml --check --diff
   ```
   Review the predicted changes. They should be limited to the three tasks above.

> **STOP — Verification Gate G1.**
> Confirm the `--check` output proposes only the intended changes (install nginx, start/enable
> service, enable HTTP). If it proposes anything else, stop and investigate before applying.

2. **Apply for real:**
   ```
   ansible-playbook -i inventory install-nginx.yml
   ```

---

## 5. Post-Change Verification

Run against the target and confirm each success criterion:

| Criterion | Command | Expected PASS result |
|-----------|---------|----------------------|
| Installed | `ansible <host> -b -m command -a 'rpm -q nginx'` | Installed version string |
| Active | `ansible <host> -b -m command -a 'systemctl is-active nginx'` | `active` |
| Enabled | `ansible <host> -b -m command -a 'systemctl is-enabled nginx'` | `enabled` |
| HTTP 200 | `ansible <host> -b -m shell -a 'curl -s -o /dev/null -w "%{http_code}" http://localhost/'` | `200` |
| Firewall | `ansible <host> -b -m command -a 'firewall-cmd --list-services'` | Includes `http` |

> **STOP — Verification Gate G2 (idempotency).**
> Re-run `ansible-playbook -i inventory install-nginx.yml` (and/or `--check`). Every task must
> report `ok` with `changed=0`. A non-zero `changed` count on a second run means the playbook is
> not idempotent — stop and investigate before declaring the task done.

---

## 6. Rollback Plan (Mandatory)

The change is small and cleanly reversible. Rollback removes exactly what the playbook added and
nothing more.

### Option A — Counter-playbook (preferred, auditable)
Save as `rollback-nginx.yml` and run with the same dry-run-first discipline
(`ansible-playbook -i inventory rollback-nginx.yml --check --diff`, then apply):

```yaml
---
- name: Roll back nginx installation on RHEL
  hosts: "{{ target_hosts | default('web') }}"
  become: true
  tasks:
    - name: Stop and disable nginx service
      ansible.builtin.service:
        name: nginx
        state: stopped
        enabled: false
      failed_when: false

    - name: Remove HTTP from firewalld
      ansible.posix.firewalld:
        service: http
        permanent: true
        immediate: true
        state: disabled

    - name: Remove the nginx package
      ansible.builtin.dnf:
        name: nginx
        state: absent
```

### Option B — Manual fallback (if Ansible is unavailable mid-incident)
Run directly on the target host:
```
sudo systemctl stop nginx && sudo systemctl disable nginx
sudo firewall-cmd --permanent --remove-service=http && sudo firewall-cmd --reload
sudo dnf remove -y nginx
```

### Rollback verification
- `rpm -q nginx` returns "package nginx is not installed".
- `firewall-cmd --list-services` no longer includes `http`.
- `ss -tlnp | grep ":80 "` returns nothing (port 80 free again).

### Notes and safeguards
- Removing the `nginx` package leaves the default `/etc/nginx` config untouched only if it was not
  package-owned; since this example installs the stock package with no custom config, no
  pre-change config backup is required. If the playbook is later extended to deploy custom config,
  add a task to back up `/etc/nginx` before changes and restore it here.
- The pre-check that port 80 was `FREE` (P5/G0) guarantees rollback does not disturb a pre-existing
  service on that port.

---

## 7. Post-Task Actions

- Capture verification-gate output (G0–G2) as completion evidence.
- Commit the playbook (and rollback playbook) to version control for auditability.
- Log any unrelated findings (e.g., weak SSH ciphers observed during connection, unexpected
  listening services) to `ai-backlog.md` rather than acting on them now.
