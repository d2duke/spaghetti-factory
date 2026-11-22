# Log

Newest first. Each entry: context, what happened, lessons (linked to [BEST-PRACTICES.md](BEST-PRACTICES.md)), evidence.

---

## 2026-11-21 (evening) — Portable review skill

Saved the `project-review` skill to [`skills/project-review/`](skills/project-review/) in two forms: `SKILL.md` (the Claude version, copied from the saved skill) and `PORTABLE.md` (a paste-ready block for other AI tools, with raw links to the latest guide). **Lesson:** keep tool-specific wrappers thin and point them at one shared source (`REVIEW-GUIDE.md`), so switching AI tools never forks the knowledge.

---

## 2026-11-21 (evening) — A review guide for future reviews

**Context.** The playbook says how to build. Reviews need something different: a list of the pitfalls AI-written code and agent setups commonly hit, each with a concrete check, that a reviewing agent reads *before* looking at a project.

**What we did.** Added [`REVIEW-GUIDE.md`](REVIEW-GUIDE.md): ground rules, a 7-step procedure, a 42-item catalog in six areas (verifying state, AI-written code smells, agent architecture, GitHub Actions security, data and integrations, docs and status) and a report format. Items are tagged **[field]** when they're widely documented elsewhere (reward-hacked tests, slopsquatting, the lethal trifecta, Actions script injection and unpinned actions, bloated instruction files) and **[ours]** when they come from this log.

**Lessons.**
1. A good review guide is a checklist of *checks*, not advice. Every item names a command or something specific to look at.
2. Reviewers over-report when asked for gaps. The guide limits findings to correctness, security, data loss and missed requirements.
3. Check a repo's visibility before publishing to it. This repo is public, so the first log entry was rewritten without private project details. → `E5`

---

## 2026-11-21 — Reviewing a side project's build workflow; designing an agent crew; repo cleanup

**Context.** A private side project (a TypeScript data pipeline run with GitHub Actions) had grown an "architect" planner (an LLM call in GitHub Actions that writes a worker brief) and a "durable claim" system (immutable Git tags as task locks). The goal was a full-circle, automated loop with a coding agent. I wanted named agents, a way to *see* them work, and something reusable across projects instead of rebuilding each time.

**What we found.**
- Strong foundations: human approval gates, evidence-based status words, issues as handoffs, a planner that can say "hold."
- The loop wasn't closed. The brief → claim → coding handoff was still manual.
- The claim system (~110 lines plus race tests) had never been used: zero claim tags existed. → 1.7
- No `AGENTS.md`/`CLAUDE.md`, an empty README, and 25 docs. Three long docs were pasted into every planner prompt. → 2.1, 2.2
- 13 branches: 7 merged leftovers, 3 superseded (their content had already reached `main` through a later PR), 1 holding unique notes, and 2 with open PRs. Stale branches made the planner see false overlaps. → 3.2
- Two workflows committed reports to `main` after every merge (25 of 131 commits), so every branch fell behind immediately. → 3.5

**What we built.**
- **Agent Crew Kit** (reusable): three named agents (🧭 Atlas plans, 🔨 Forge builds, 🛡️ Sentinel reviews) as Claude Code subagent files, label-driven GitHub workflows on `anthropics/claude-code-action`, an `AGENTS.md` template, an issue template, a label setup script, an installer (`./install.sh <repo>` or `--global`), a Mission Control dashboard (live or demo) and a terminal status view. Tested: dashboard in demo and live modes with real repo data at phone and desktop widths, unit tests for the status logic, the installer on a scratch copy, the planner's snapshot step against the real repo. **Not yet run live in GitHub Actions.**
- **Project cleanup PR:** workflows publish to run summaries and artifacts instead of committing, the data-collection job carries its snapshot between runs as an artifact (with a committed baseline as fallback), parked notes from a leftover branch moved into `main`, a stale doc link was fixed, and the handoff doc gained an update log plus an outstanding-actions checklist.
- Ten stale branches deleted, with their last SHAs recorded in the cleanup PR. Auto-delete of head branches turned on.

**Lessons.** 1.1–1.15, 2.1–2.5, 3.1–3.8, 4.1–4.4. The biggest ones:
1. Before building orchestration, look for the off-the-shelf version. The missing "dispatch a coding agent" step already exists as a GitHub Action. (1.13)
2. Build for the concurrency you have, not the concurrency you imagine. (1.7)
3. Verify state after every destructive command. A shallow clone gave wrong counts, and a delete script reported success while every push failed. (3.3, 3.4)
4. Generated reports belong in Actions artifacts, not in commits to `main`. (3.5)

**Open threads.**
- Merge the cleanup PR, delete the last leftover branch, and close out the two open PRs (tracked in that project's milestone checklist).
- Then install the agent crew: retire the custom planner and claim workflows, remove the planner's now-unused API key secret, and close the planner's tracking issue.
- Watch the first live crew runs and log what breaks.
