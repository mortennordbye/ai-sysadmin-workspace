# CLAUDE.md

Behavioral guidelines to reduce common LLM coding and systems administration mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward extreme caution, system stability, and verification over speed. For trivial tasks, use judgment.

## 0. Environment & Tech Stack

**Know where you are operating.**
- **Infrastructure:** VMware-based virtualized platform.
- **Operating Systems:** Windows Server and Linux RHEL exclusively. Do not provide Debian/Ubuntu or alternative hypervisor commands unless explicitly requested.

## 1. Think Before Executing

**Don't assume. Don't hide confusion. Surface tradeoffs and blast radius.**

Before implementing or proposing any changes:
- State your assumptions explicitly regarding the current server state or VMware configuration. If uncertain, ask.
- If multiple interpretations of a fix exist, present them—don't pick silently.
- **Mandatory:** No system changes are to be executed without formulating a concrete, step-by-step rollback plan first.
- If a simpler administrative or automation approach exists, say so. Push back when warranted.
- If a shell command throws a permission error or an unexpected state occurs, stop. Name what is confusing. Ask. Do not automatically attempt sudo or administrative overrides.

## 2. Simplicity First

**Minimum configuration that solves the problem. Nothing speculative.**

- No features, firewall rules, or installed packages beyond what was explicitly asked.
- No abstractions for single-use automation or playbooks.
- No "flexibility" or "configurability" that wasn't requested.
- If you write a 200-line script and it could be accomplished with a simple native utility or a short Ansible task, rewrite it.

Ask yourself: "Would a senior infrastructure engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes & Scope Management

**Touch only what you must. Clean up your own mess. Use the Backlog.**

When editing existing configurations, registry keys, or scripts:
- Don't "improve" adjacent configurations, comments, or formatting.
- Don't refactor systems that are not broken.
- Match existing style, documentation standards, and enterprise naming conventions.
- **The Backlog Rule:** If you notice unrelated misconfigurations, dead code, or security flaws, do not attempt to fix them now. Mention them to the user and append them to `ai-backlog.md`.

When your changes create orphans:
- Remove variables, temporary directories, or configurations that your changes made unused.
- Don't remove pre-existing dead components unless asked.

The test: Every changed line or executed command should trace directly to the user's immediate request.

## 4. Goal-Driven Execution & Context Handovers

**Define success criteria. Plan first. Loop until verified.**

Transform tasks into verifiable goals:
- "Fix the network route" → "Generate a plan, apply the persistent route, verify connectivity via ping/traceroute, then test the fallback path."
- "Update configuration" → "Ensure dry-runs or ansible-playbook --check passes before and after applying the change."

For multi-step tasks, state a brief plan and wait for explicit human approval before execution: