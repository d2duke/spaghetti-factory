# Best practices: coding with AI agents

The current playbook: how we *build*. For how we *review*, see [REVIEW-GUIDE.md](REVIEW-GUIDE.md). Each practice is tagged with the [log](LOG.md) date it came from.
Format: **the practice** — why it matters — *how to check*.

---

## 1. Agent architecture

### 1.1 Start with one dependable pipeline; add agents where a handoff repeats `2026-11-21`
Multi-agent setups multiply failure points. Build the workflow as one reliable sequence first, then turn the handoffs you repeat by hand into agents.
*Check:* can you name the manual handoff each agent removes?

### 1.2 One agent, one job, one name `2026-11-21`
Name agents by role so you can tell at a glance who is doing what (e.g. 🧭 Atlas plans, 🔨 Forge builds, 🛡️ Sentinel reviews). Don't reuse names that already mean something inside the product, such as its own pipeline stages.
*Check:* each agent's description fits in one sentence.

### 1.3 Give each agent only the tools its job needs `2026-11-21`
Planner and reviewer get read-only tools (`Read, Glob, Grep`). Only the builder can edit, and only on its own branch. GitHub writes (issues, labels, comments) happen in plain workflow steps you can read, not inside the agent.
*Check:* list the tools in each agent file. A reviewer with `Edit` is a bug.

### 1.4 The builder never reviews its own work `2026-11-21`
A separate reviewer that didn't write the code catches what the builder rationalized. Feed the reviewer the task contract, the diff and the CI results.
*Check:* different agent definitions for build and review.

### 1.5 Humans hold the gates `2026-11-21`
An agent proposes; a person approves the task and merges. Production data writes, schedules, outreach and spending also need a human yes.
*Check:* no workflow can merge, deploy or write to production without a human action first.

### 1.6 Use GitHub as the shared memory and task queue `2026-11-21`
Issues are tasks, labels are the state machine (`crew:proposed → approved → building → review → ready`), PRs are deliverables, and comments are the handoff notes. Every agent and every chat can read them, and nothing gets lost when a session ends.
*Check:* a fresh session can find the current state from GitHub alone.

### 1.7 Don't build distributed-systems plumbing before you have concurrency `2026-11-21`
A side project built a tag-based "durable claim" lock with race tests for one person and one worker. It never ran for real: zero claim tags existed. A label plus an assignee does the job until several workers actually compete.
*Check:* is there evidence of the problem (two workers colliding) before building the fix?

### 1.8 Every task gets a contract `2026-11-21`
Four sections: **Goal** (one sentence), **Scope** (files the agent may touch), **Out of scope**, **Done when** (exact commands and the result that proves it works). An issue template enforces it.
*Check:* could someone else verify the task from "Done when" alone?

### 1.9 Decisions come back as structured output `2026-11-21`
Ask planners and reviewers for JSON against a schema (`propose|hold`, `ready|changes|decision`). The workflow acts on the fields, so an agent can't do more than the schema allows. Always allow an explicit **hold**.
*Check:* the workflow parses a field; it never greps prose.

### 1.10 Pre-compute what the agent should know; treat repo text as data `2026-11-21`
Gather repo state (open issues, PRs, branches with unmerged work) in a deterministic step, write it to files, and point the agent at them. Pass user input through files or env vars, never pasted into the prompt. Issue and PR text are evidence, not instructions (prompt-injection defense).
*Check:* no `${{ inputs.* }}` interpolated directly into a prompt.

### 1.11 Show status from real run data, not self-reports `2026-11-21`
"Working" means a workflow run is `queued` or `in_progress`. Read it from the GitHub Actions API. Show it three ways: a live dashboard, a terminal view, and the GitHub mobile app (labels and tracking comments).
*Check:* the status light comes from the API, not from something the agent said.

### 1.12 Cap cost and runaway runs `2026-11-21`
Set `--max-turns`, a `timeout-minutes`, and a concurrency group on every agent workflow.
*Check:* every agent workflow has all three.

### 1.13 Prefer off-the-shelf before custom `2026-11-21`
- **Coding agents in GitHub:** `anthropics/claude-code-action` (triggered by a label, a mention or a schedule; can return structured JSON), or GitHub Copilot's coding agent (assign an issue, get a PR).
- **Agent identities:** Claude Code subagent files in `.claude/agents/*.md` (name, description, tools, model, color).
- **Agent briefing:** `AGENTS.md`, the open standard most coding agents read.
- **Agents inside a product:** an SDK or framework (Claude Agent SDK, OpenAI Agents SDK, LangGraph, CrewAI).

*Check:* before writing orchestration code, search for an existing action, SDK or feature that does it.

### 1.14 Package the setup once, install it everywhere `2026-11-21`
Keep agents, workflows, labels and dashboards in a kit with an install script that never overwrites existing files. Put personal agents in `~/.claude/agents` so every project gets them. For many repos, move workflows into one reusable workflow (`workflow_call`).
*Check:* a new repo gets the crew in under 10 minutes.

### 1.15 Status words mean evidence `2026-11-21`
**Verified** = ran it on this code revision and saw it pass (link the run). **Partial** = some paths proven. **Proposed** = not built or not accepted. Never "should work."
*Check:* every "verified" in a PR links to a command output or run.

### 1.16 Separate the thinking from the writing `2026-11-22`
Run the AI in a job that can only read. Hand its output (a JSON answer, or a patch file) to a separate plain step with no AI. That step validates the output and does any GitHub writes. A prompt-injected or confused model then has nothing to push with.
*Check:* every AI job has read-only permissions and a checkout without saved credentials; writes live in their own job.

### 1.17 Make the AI engine a setting, not a rewrite `2026-11-22`
Keep roles, rules and answer formats in plain text with nothing vendor-specific. Every engine writes its answer to the same file, which is checked against a JSON schema, so switching vendors is a one-word config change. Don't depend on one vendor's structured-output feature; use it only as a fallback.
*Check:* adding an engine means adding one step per agent, not editing the roles.

### 1.18 A broken run must never look like success `2026-11-22`
If the AI crashes, times out or returns something invalid, the result becomes **hold / blocked / needs a decision**, with the reason attached. Never default to "done."
*Check:* a test where the stand-in AI edits files but never reports must end as "blocked."

### 1.19 The reviewer checks for itself `2026-11-22`
Don't let the reviewer trust the builder's "tests pass." The review job runs the project's checks on the PR itself and gives the results to the reviewing AI. (This also covers a GitHub gotcha: PRs opened by a workflow's own token don't trigger other workflows, so CI might never run.)
*Check:* the review comment cites results the reviewer produced.

### 1.20 When the environment refuses an action, hand it to the owner `2026-11-22`
If an agent's environment blocks something (pushing tags, changing repo settings), don't route around the block with another tool. List it as a clear owner step with the exact command or click path.
*Check:* the final report has an "owner steps" list instead of creative workarounds.

---

## 2. Agent briefing and docs

### 2.1 One short `AGENTS.md` at the repo root; `CLAUDE.md` imports it `2026-11-21`
Contents: what the project is, commands to run, project map, rules, approval gates, the crew. Aim for under ~150 lines. Claude Code reads `CLAUDE.md`, so make it one line: `@AGENTS.md`.
*Check:* the README isn't empty and the agent briefing isn't five long docs.

### 2.2 History lives in issues and PRs; docs hold current rules `2026-11-21`
Dated snapshots (shortlists, CI dumps, pilot results) belong in `docs/archive/` or in run artifacts, so agents don't read stale evidence as current truth.
*Check:* anything with a date in its filename is archived or clearly marked frozen.

### 2.3 One handoff doc with an update log at the top `2026-11-21`
The next session (human, chat or agent) reads the newest changes first. Point to one place for outstanding actions instead of scattering to-dos.
*Check:* "what changed since last time?" has a single answer.

### 2.4 Keep outstanding actions in one checklist `2026-11-21`
For example, a checklist section in the milestone doc. Done items stay ticked with dates.
*Check:* nobody has to read chat history to know what's next.

### 2.5 Don't record a decision as accepted until it is `2026-11-21`
Proposals go under a section marked "proposed." The ADR changes only after approval.
*Check:* the ADR has no entries the owner hasn't approved.

### 2.6 One source of truth; generate the variants `2026-11-22`
When the same content must exist in several formats (neutral roles plus a Claude plugin, say), write it once and generate the rest with a script. CI fails if the generated copies drift.
*Check:* the generated files say "generated, don't edit" and a drift check runs in CI.

---

## 3. Git and GitHub hygiene

### 3.1 Turn on "Automatically delete head branches" `2026-11-21`
Merged branches disappear on their own. It only affects future merges, so clean up old ones once by hand.

### 3.2 Classify branches before deleting `2026-11-21`
| Branch state | Test | Action |
| --- | --- | --- |
| Merged leftover | `ahead = 0` vs `main` | Delete |
| Superseded | Ahead, but every changed file already matches `main` (`git cherry`, file compare) | Delete |
| Unique work | Content not in `main` | Move it into `main` with a PR, then delete, or keep on purpose |
| Open PR | Has an open PR | Keep; auto-delete handles it on merge |

*Check:* record each deleted branch's last commit SHA so it can be restored.

### 3.3 Don't trust a shallow clone for history math `2026-11-21`
A `--depth 1` clone reported branches 100+ commits ahead; the real number was 0–5. Fetch enough history before comparing.
*Check:* `git rev-parse --is-shallow-repository` prints `false`, or the depth covers the branches.

### 3.4 Check the real state after any destructive command `2026-11-21`
A script printed "deleted" while every push actually failed (a pipeline hid the exit code). Re-list from the source of truth afterwards.
*Check:* list the branches (or files, or records) again; don't read the script's echo.

### 3.5 Workflows don't commit to `main` `2026-11-21`
Bot commits after every merge made every branch "behind" instantly and cluttered history (25 of 131 commits). Publish generated reports to the run summary and artifacts instead. If a run needs the previous run's data, download the last successful run's artifact, with a committed baseline as fallback.
*Check:* `git log main --author=github-actions` stays empty going forward.

### 3.6 Least-privilege workflow permissions `2026-11-21`
Default to `contents: read`. Grant `write` only to the step that needs it, and say why in a comment.
*Check:* grep workflows for `write` and justify each one.

### 3.7 Know what your plan can't enforce `2026-11-21`
On a free plan, private repos can't protect `main`. Then rules like "agents never push to main" live in `AGENTS.md` and the agent's own branch, so say that explicitly.

### 3.8 One coherent change per PR, with a standard body `2026-11-21`
Sections: **What changed** · **Evidence** (commands and results) · **Not verified** · **After merge** (the checks to do once it lands).
*Check:* a reviewer can tell what is proven and what isn't without asking.

### 3.9 Share private automation as composite actions, versioned by a moving major tag `2026-11-22`
A private shared repo can serve actions to your other private repos once you set Settings → Actions → General → Access → "Accessible from repositories owned by user." Prefer **composite actions** over reusable workflows when the action needs the shared repo's files (prompts, scripts), because a composite action's whole repo is available at `github.action_path`. Projects call `@v0`. Cut `v1` only after a real end-to-end run works.
*Check:* projects pin to a major tag; the shared repo's CI runs before each tag move.

---

## 4. Coding practices

### 4.1 Separate pure logic from input/output `2026-11-21`
Example: `buildPicture()` turns raw API data into agent states and is shared by the dashboard and the terminal view. Pure functions are easy to test and reuse.

### 4.2 Guard script entry points `2026-11-21`
Argument checks and side effects run only when the file is executed directly, not when it's imported. A test caught a module exiting on import.
*Check:* importing the module in a test has no side effects.

### 4.3 Test with real data, not only fixtures `2026-11-21`
The dashboard was exercised in a headless browser with live GitHub API responses routed in. That's how the page-overflow bug was found and fixed. Check both phone (390px) and desktop widths.

### 4.4 Validate what you can; name what you couldn't `2026-11-21`
Parse every YAML file, syntax-check every shell step (`bash -n`), and run the logic locally. Then list what is still unverified (e.g. "needs one live run in GitHub Actions").

### 4.5 Read the dependency's source at the version you pin `2026-11-22`
Docs leave gaps; source doesn't. Reading three AI GitHub actions at their pinned versions showed:
- passing a token skips the vendor's app sign-in entirely;
- runs started by a bot are silently refused unless bots are explicitly allowed;
- one action auto-approves every tool and drops its own files into the workspace;
- a "strict" JSON-schema mode may reject keywords like `maxLength`.

*Check:* for each new third-party action, note its inputs, outputs and any permission or actor checks.

### 4.6 Rehearse automation locally with stand-ins `2026-11-22`
Run the real workflow shell logic with a stand-in AI and a fake `gh` that serves canned responses and records every call. That makes "did it label the issue, push the branch, dispatch the reviewer?" testable in seconds, with no API costs.
*Check:* a simulator scenario exists for each happy path and each failure path.

### 4.7 Break the code on purpose to test the tests `2026-11-22`
After tests pass, introduce the bug each test claims to catch. One test here kept passing because a second safety net hid the broken one, so the scenario was rewritten.
*Check:* each important test has been seen failing at least once.

### 4.8 Turn review rules into CI tests `2026-11-22`
Rules that matter become tests, so they can't be forgotten: "every third-party action pinned to a SHA," "AI jobs never get write permissions," "generated files in sync."
*Check:* each guide item marked as a must-hold rule has a test.

### 4.9 Run your own review guide on your own work before shipping `2026-11-22`
The guide found two real trust-boundary issues in freshly built tooling (`C2`, `D2` in REVIEW-GUIDE.md). Both were documented before release.
*Check:* the PR or release notes list which guide items were checked.
