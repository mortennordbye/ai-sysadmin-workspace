# secrets/ — GIT-IGNORED

Everything in this directory **except this README** is ignored by git (see the top-level
`.gitignore`). It exists so there is an obvious, local-only place to stage sensitive material
*temporarily* during a task — but it is **not** a vault and must never be committed.

## Rules

- **Never commit secrets to this repository.** Not here, not in `inventory.md`, not in task docs,
  not in handovers. This repo is for declarative plans and state, not credentials.
- Keep real secrets in a proper secret store: HashiCorp Vault, Ansible Vault, Windows Credential
  Manager / LAPS, or your organization's approved vault.
- If you must stage a file locally (e.g., an exported config to diff), put it here so `.gitignore`
  prevents an accidental commit, and delete it when the task is done.
- If you ever discover a secret that *was* committed, treat it as compromised: rotate it and purge
  it from history. Do not just delete the file in a later commit.
