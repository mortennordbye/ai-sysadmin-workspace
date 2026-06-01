# Target State: Install nginx on RHEL via an Ansible Playbook

**Task ID:** example-nginx-ansible
**Platform:** Red Hat Enterprise Linux (VMware vSphere virtualized)
**Document type:** Declarative target state (the "what", not the "how")
**Status:** Example / template

---

## 1. Objective

Install and enable the nginx web server on a target RHEL host using a single, idempotent Ansible
playbook executed from an Ansible control node. On completion, nginx must be installed, running,
enabled at boot, and serving its default page over HTTP, with the host firewall permitting that
traffic. The playbook must be safe to re-run: a second execution must report zero changes.

This is a deliberately small example chosen to demonstrate the workspace's dry-run and
idempotency discipline, not to model a production web platform.

---

## 2. Scope

**In scope:**
- Installing the `nginx` package on the target RHEL host via the system package manager (`dnf`),
  orchestrated by Ansible.
- Ensuring the `nginx` service is started and enabled.
- Opening the HTTP service in `firewalld` so the default page is reachable.
- Confirming idempotency of the playbook.

**Explicitly out of scope (defer to `ai-backlog.md` if discovered):**
- TLS/HTTPS configuration, certificates, or port 443.
- Custom virtual hosts, reverse-proxy configuration, or application content.
- SELinux policy changes beyond the default permitted by the package (note in backlog if a custom
  port or non-standard document root later requires an SELinux adjustment).
- Hardening of cipher suites, headers, or fleet-wide nginx baselines.

---

## 3. Assumptions

Confirm before planning execution. If any is false, stop and revise.

1. The target host is a supported RHEL version with a valid subscription (or access to a local
   repository/satellite) that provides the `nginx` package.
2. An Ansible control node is available and already has SSH key-based access to the target host
   with an account permitted to `become` (escalate to root via sudo).
3. The target host is reachable on the management network and is represented in the Ansible
   inventory.
4. `firewalld` is the active host firewall on the target (the standard RHEL default).
5. Port 80/tcp is not already bound by another service on the target host.

---

## 4. Pre-Requisite Conditions

All must be **true and verified** before any change is executed.

| # | Pre-requisite | How it is confirmed |
|---|---------------|---------------------|
| P1 | Ansible is installed on the control node | `ansible --version` returns a supported version |
| P2 | The target host is in inventory | Host appears under the intended group in the inventory file |
| P3 | Connectivity and privilege escalation work | `ansible <host> -m ping` succeeds; `ansible <host> -b -m command -a 'id'` returns `uid=0` context |
| P4 | The `nginx` package is resolvable | `ansible <host> -b -m command -a 'dnf info nginx'` shows an available package |
| P5 | Port 80 is free on the target | `ansible <host> -b -m shell -a 'ss -tlnp | grep ":80 " || echo FREE'` returns `FREE` |

---

## 5. Success Criteria

The task is successful only when **every** criterion is met and verified by a human administrator:

1. **Package installed.** `rpm -q nginx` on the target returns an installed version.
2. **Service running and enabled.** `systemctl is-active nginx` returns `active` and
   `systemctl is-enabled nginx` returns `enabled`.
3. **HTTP responds.** `curl -s -o /dev/null -w "%{http_code}" http://localhost/` on the target
   returns `200`.
4. **Firewall permits HTTP.** `firewall-cmd --list-services` includes `http` (permanently, so it
   survives reload).
5. **Playbook is idempotent.** A second run of `ansible-playbook ... --check` (and a real second
   run) reports `changed=0` for all tasks.

---

## 6. Definition of Done

- All success criteria in Section 5 are met and the verifying output is captured as evidence.
- The playbook lives under version control (e.g., in `templates/` or the task directory) so the
  change is reproducible and auditable.
- Any unrelated issues observed (e.g., weak SSH ciphers, unexpected listening services) are logged
  to `ai-backlog.md` rather than fixed in flight.
- The `approved-plan.md` was reviewed and approved by a human before execution, and a `--check`
  dry run was performed before applying real changes.
