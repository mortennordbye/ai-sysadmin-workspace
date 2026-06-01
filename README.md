# AI Sysadmin Workspace

**A declarative AI "Headquarters" template for Cloud Engineers and Systems Administrators.**

This repository serves as a blueprint for safely integrating advanced AI assistants (like Gemini CLI, Claude Pro, or Cursor) into your daily IT infrastructure workflows. It shifts the paradigm from using AI as a glorified "chatbot" to treating it as an integrated, strictly governed **Co-Pilot**.

## This Is a Starter Template — Make It Your Own

This repo is a **scaffold**, not a place to store your work. It lives on GitHub only to give you a
clean starting point for building your **own** private workspace — the kind you need when you
administer many different systems rather than living inside one mono-repo.

- **Get it:** click **Use this template** on GitHub (or clone) to create your own copy.
- **Make it yours:** your copy is *yours*. Fill in real clients, inventories, tasks, and handovers
  locally — keep it private (a private repo or just a local folder).
- **Don't push real data back here:** the public template carries only the rules, the worked
  examples, and empty scaffolding. Real hostnames, inventories, and especially **secrets** stay in
  your private space. The included `.gitignore` is there to protect *your* copy from accidental
  commits of credentials — secrets belong in a vault, never in any repo.
- **`Client-Alpha/` is a sample.** Use it as a model, then replace or delete it. Onboard a real
  client by copying `customers/Client-Alpha/` and renaming.

## The Philosophy: Declarative AI Orchestration

Traditional "imperative" AI usage involves copying and pasting bash snippets back and forth, leading to context loss, hallucinations, and manual errors. 

This workspace is built on **Declarative AI Orchestration**:
1. You define the **Desired State** and the **Environmental Constraints**.
2. The AI generates a comprehensive, step-by-step execution blueprint.
3. You review, approve, and execute.

## Core Features & Guardrails

This template injects persistent, strict rules into your AI's context window. By using the included blueprint file (`GEMINI.md` / `CLAUDE.md` / `.cursorrules`), your AI is automatically bound by the following enterprise guardrails:

* **The "Plan First" Rule:** The AI is instructed *never* to execute terminal commands or provide configuration files without first generating a step-by-step plan for your approval.
* **Mandatory Rollbacks:** Every AI-generated plan must include explicit rollback instructions (e.g., how to revert a firewall rule or restore a backup) before proceeding.
* **Zero Scope Creep:** The AI is trained to log unrelated infrastructure bugs it discovers into `ai-backlog.md` rather than getting distracted from the primary task.
* **Context Resets:** Frameworks for utilizing `handover.md` documents to cleanly reset the AI's memory window when chats get too long, eliminating hallucinations.

## Repository Structure

This workspace is built for administrators who manage **many environments, not a single repo**.
Live work is organized per client; the top-level `tasks/` holds only reusable examples.

```text
ai-sysadmin-workspace/
├── README.md                 # This file — philosophy, structure, and usage
├── STATUS.md                 # Cross-client radar: every task in flight, at a glance
├── GEMINI.md                 # Global AI rules (auto-injected for Gemini)
├── CLAUDE.md                 # Global AI rules (auto-injected for Claude)
├── .cursorrules              # Global AI rules (auto-injected for Cursor)
├── .gitignore                # Keeps secrets and noise out of version control
├── ai-backlog.md             # Deferred technical debt and side findings logged by the AI
├── templates/                # Reusable, safe starting points (base configs, playbook/doc stubs)
│   └── .gitkeep
├── tasks/                    # Reusable EXAMPLE tasks — copy these as starting points
│   ├── example-ad-migration/         # Windows: migrate AD DS to a newer Windows Server version
│   │   ├── target-state.md           #   declarative goal + success criteria
│   │   └── approved-plan.md          #   reviewed plan with verification gates + rollback
│   └── example-nginx-ansible/        # RHEL: install nginx via an Ansible playbook
│       ├── target-state.md
│       └── approved-plan.md
└── customers/                # One directory per client/environment — where LIVE work happens
    └── Client-Alpha/
        ├── README.md                 # Engagement context for this client
        ├── inventory.md              # Source of truth: systems in scope (hosts, roles, owners)
        ├── active-handover.md        # Context-reset checkpoint (see "Resetting Context" below)
        ├── tasks/                    # Live tasks for this client (one dir per task)
        │   └── .gitkeep
        └── secrets/                  # GIT-IGNORED — never commit credentials; use a vault
            └── README.md
```

The three rule files (`GEMINI.md`, `CLAUDE.md`, `.cursorrules`) carry the **same** guardrails for
different AI tools. Keep whichever your team uses; they are interchangeable in intent.

To onboard a new client, copy `customers/Client-Alpha/` and rename it. Keep secrets out of the repo
entirely — `inventory.md` and the task/handover files hold identifiers and state, never credentials.

## How to Use This Template

This repository is a **static template** — there is no install script. Create your own copy from it,
keep that copy private, then follow the declarative workflow below for each piece of work.

### Getting Started Safely (keep secrets off GitHub)

The single most important rule: **the copy you fill with real data must not have a public GitHub
remote.** Pick one of these two safe setups before you add any client information.

**Option A — Private GitHub repo (you want version history in the cloud).**
1. On the template's GitHub page, click **Use this template → Create a new repository**.
2. Set visibility to **Private**. Create it.
3. Clone *your private repo* (not the public template) and work there.

**Option B — Local only (nothing goes to GitHub at all).** Clone the template, then detach its
remote so an accidental `git push` has nowhere to go:
```bash
git clone https://github.com/<owner>/ai-sysadmin-workspace.git my-workspace
cd my-workspace
git remote remove origin          # now there is no remote to push secrets to
# optional: point it at your OWN private remote later with `git remote add origin <private-url>`
```

**Safety nets that apply either way:**
- The bundled `.gitignore` already excludes `**/secrets/*` and common credential patterns
  (`*.key`, `*.pem`, `.env`, `credentials*`, `vault-password*`, …). Keep it.
- **Never paste secrets into markdown** — not in `inventory.md`, task docs, or handovers. Those
  hold identifiers and state only. Real credentials live in a vault (Ansible Vault, HashiCorp
  Vault, Windows Credential Manager / LAPS).
- **Check before every commit:** run `git status` and confirm no credential files or `secrets/`
  contents are staged. To prove the ignore rules work: `git check-ignore -v customers/*/secrets/test.key`
  should report a match.
- If a secret ever does get committed, treat it as compromised — **rotate it** and purge it from
  history; don't just delete the file in a later commit.

### Workflow

1. **Make your own copy and point your AI at the rules.** Using a safe setup above, open your
   private workspace in your AI-enabled tool (Gemini CLI, Claude, or Cursor). The matching rule
   file is auto-injected, binding the assistant to the Plan-First, mandatory-rollback, and
   surgical-scope guardrails.

2. **Pick the client and confirm the inventory.** Work happens under `customers/<client>/`. Copy
   `customers/Client-Alpha/` to onboard a new one, and make sure `inventory.md` accurately lists the
   systems in scope — every plan relies on it.

3. **Define the desired state.** Create a task directory, `customers/<client>/tasks/<your-task-name>/`,
   and write a `target-state.md` describing the *outcome* you want: the objective, scope, assumptions,
   pre-requisite conditions, and **verifiable success criteria**. Use the two top-level `tasks/example-*`
   directories as models — copy one as a starting point. Add the task to `STATUS.md`.

4. **Let the AI produce the plan.** Ask the assistant to generate `approved-plan.md` satisfying your
   target state. A valid plan states its assumptions and blast radius, lists pre-change checks,
   breaks the work into steps separated by **human verification gates**, and ends with a concrete,
   phase-by-phase **Rollback Plan**.

5. **Review, approve, execute.** Read the plan. The AI must not execute anything until you approve.
   Run the steps yourself (or have the AI run them where appropriate), pausing at each verification
   gate to confirm the expected result before continuing. Always run dry-runs first where the tool
   supports them (`ansible-playbook --check`, etc.). Keep `STATUS.md` current as phases complete.

6. **Log side findings — don't chase them.** If you or the AI notice unrelated misconfigurations,
   security gaps, or technical debt, append them to `ai-backlog.md` and keep going. Scope creep is
   the enemy of a safe change.

7. **Reset context when the session degrades.** If the chat window grows bloated or the assistant
   starts looping on an error, stop. Capture the exact machine state, completed steps, remaining
   tasks, and current rollback position in `customers/<client>/active-handover.md`, then start a
   **fresh** session and re-seed it from that single document. See `customers/Client-Alpha/active-handover.md`
   for the blueprint.

### The Core Loop

```text
target-state.md  →  approved-plan.md  →  human review  →  execute w/ verification gates
        ↑                                                          │
        └──────────── handover.md (reset context) ←── backlog (log side findings) ←──┘
```
