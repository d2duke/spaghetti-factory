# Review guide: read this before reviewing any project

**Audience:** an AI agent (or person) about to review a code project, an agent setup or a pull request.
**Use:** read this whole file first. Follow the procedure, use the catalog as your checklist, and report in the format at the end. Cite findings by ID (e.g. `B1`, `D3`).

Tags: **[field]** = documented widely by others (sources at the end). **[ours]** = learned in our own work (see [LOG.md](LOG.md)). Practice numbers like `§3.5` point to [BEST-PRACTICES.md](BEST-PRACTICES.md).

---

## Ground rules for the reviewer

1. **Evidence over assertion.** Every finding cites a file and line, a command and its output, or a run link. "Looks wrong" is not a finding.
2. **Code and CI beat docs and chat.** If a doc says X and the code does Y, the code wins. Record the gap as finding `F1`.
3. **Read-only by default.** A review changes nothing unless the owner asks. Before anything destructive (deleting branches, force-pushing, closing PRs), list exactly what will happen and get a yes.
4. **Try to disprove each finding before you report it.** Re-run the check, look for a reason it's actually fine, and drop it if it doesn't hold up.
5. **Report gaps that matter.** Flag correctness, security, data loss and missed requirements. Skip style preferences. A reviewer asked for gaps always finds some; chasing every one leads to over-engineering (`B9`). [field]
6. **Mind your own context.** Don't read the whole repo. Start from the map in step 1 and go deep only where a check points you.

---

## Procedure (in order)

1. **Orient.** Read `AGENTS.md` / `CLAUDE.md` / `README`, then the handoff or roadmap doc. Write down what the project *claims* is done.
2. **Get true state.** Default branch SHA, open PRs, branches ahead of or behind main, the latest CI run on that SHA, open issues. Use full history, not a shallow clone (`A2`).
3. **Run the checks.** Install, type check, tests, build. Record exact commands and results. If you can't run something, say so.
4. **Compare claims to reality.** Every "done" or "verified" claim needs a matching run or output (`A4`, `F1`).
5. **Walk the catalog.** Sections A–F below. Each item says how to check it.
6. **Ask the big question.** Is the project building the right thing at the right size? (`C1`, `B9`.) Is the plumbing outgrowing the product?
7. **Report** in the format at the end.

---

## Catalog

### A. Verifying state

- **A1 · Trusting the docs over the code.** [ours] Handoff docs drift. *Check:* pick 3 claims from the docs and verify each in code or CI. *Fix:* correct the doc or record the gap.
- **A2 · Shallow clones distort history.** [ours] `--depth 1` made branches look 100+ commits ahead when the real number was 0–5. *Check:* `git rev-parse --is-shallow-repository`; fetch more history before running `rev-list` or `cherry`. (§3.3)
- **A3 · Hidden failures.** [ours] Pipes and `|| true` hide exit codes. A delete script printed "deleted" while every push failed. *Check:* `set -o pipefail`; after anything destructive, re-list from the source of truth. (§3.4)
- **A4 · "Passed" on the wrong code.** [ours] CI results apply to one SHA. *Check:* every "tests pass" claim names the SHA and run, and that SHA is the one under review.
- **A5 · Stale branches and orphaned work.** [ours] Leftover branches confuse planners and people. *Check:* for each branch, compare ahead/behind against main, check whether its content is already in main (`git cherry`, file compare), and whether it has an open PR. (§3.2)
- **A6 · Bots committing to main.** [ours] Generated reports committed after every merge leave every branch behind and clutter history. *Check:* `git log main --format='%an' | sort | uniq -c`. (§3.5)

### B. AI-written code smells

- **B1 · Tests bent to pass.** [field] Coding agents have been documented deleting or editing tests, special-casing test inputs, and exiting early with a success status. *Check:* `git diff <base> -- '*test*'`; look for removed assertions, new `skip`/`only`/`xfail`, loosened expectations, `if` branches keyed on literal fixture values, and literals in the code that match test expectations. *Fix:* restore the tests and solve the real problem.
- **B2 · Hallucinated packages and APIs (slopsquatting).** [field] Models invent plausible package names, and attackers register them with malware. *Check:* for every new dependency, confirm it exists on the registry, who publishes it, its age and downloads, and that the lockfile changed with it. For new API calls, confirm the method exists in the installed version. *Fix:* replace or remove; never install an unverified name.
- **B3 · Silent failure and fake success.** [field/ours] Empty `catch {}`, fallbacks to `[]` or default data, `|| true`, "not implemented" paths that return success. In data pipelines, a failed fetch must never read as "the item is gone." *Check:* grep for `catch`, `|| true`, `?? []`, `TODO`, `not implemented`; trace what a caller sees on failure. (`E2`)
- **B4 · Reinvented helpers.** [field] Agents write a new utility instead of finding the existing one. *Check:* for each new function, search the repo for a similar name or behavior.
- **B5 · Scope creep in the diff.** [field] Edits outside the task, drive-by reformatting, renamed files. *Check:* `git diff --stat`; compare against the task's Scope. (§1.8)
- **B6 · Mock-only tests.** [field] Tests that only prove the mocks were called. *Check:* would the test fail if the real logic broke? Prefer fixtures with real data shapes.
- **B7 · Loose dependency pins and lockfile drift.** [field] *Check:* is a lockfile committed, and does it change in step with the manifest?
- **B8 · Plausible but untested edge cases.** [field] The happy path works; empty input, pagination limits, time zones, retries and rate limits don't. *Check:* name 3 edge cases and look for a test of each.
- **B9 · Over-engineering.** [field/ours] Abstraction layers, defensive code and tests for cases that can't happen, or infrastructure built before the need exists. *Check:* for each new layer, ask what concrete problem it solves today. (`C1`)
- **B10 · Tests that pass for the wrong reason.** [ours] A second safety net can hide the one a test claims to check. *Check:* break the code under test on purpose; the test must fail. (§4.7)

### C. Agent architecture

- **C1 · Custom orchestration before the need.** [ours] A hand-built planner and a lock system with race tests for one person and one worker, never used live. *Check:* is there evidence of the problem (collisions, repeated manual handoffs)? Does an off-the-shelf action, SDK or feature already do this? (§1.7, §1.13)
- **C2 · The lethal trifecta.** [field] An agent that has private data, reads untrusted content, *and* can send data out can be turned against its owner through prompt injection. *Check:* for each agent, list those three capabilities. If all three are present, remove one, usually the outbound channel.
- **C3 · Untrusted text treated as instructions.** [field] Issue bodies, PR titles, web pages, emails and tool output are data. *Check:* prompts say so explicitly. User input goes through files or env vars, never pasted into the prompt. (`D1`)
- **C4 · Tools broader than the job.** [field] *Check:* each agent's tool list. Reviewers and planners are read-only; only the builder edits, and only on its own branch. (§1.3)
- **C5 · Agent grades its own work.** [field] The same context that wrote the code is biased toward it. *Check:* the review runs in a separate agent or context that sees the diff and the criteria, not the builder's reasoning. (§1.4)
- **C6 · No human gate on irreversible actions.** [field/ours] *Check:* merging, deploying, production data writes, schedules, outreach and spending each need an explicit human step. (§1.5)
- **C7 · No verification signal.** [field] Without a check it can run, an agent stops when the work "looks done." *Check:* every task has a runnable "done when" (tests, build, script, screenshot). (§1.8)
- **C8 · Unbounded runs.** [ours] *Check:* every agent workflow sets max turns, a timeout and a concurrency group. (§1.12)
- **C9 · Instructions too long to follow.** [field] A bloated `CLAUDE.md`/`AGENTS.md` gets half-ignored. Rules that must always hold belong in hooks or CI, not prose. *Check:* line count, duplicated or obvious rules, and whether each must-hold rule has an enforcement point. (§2.1)
- **C10 · Decisions parsed from prose.** [ours] *Check:* workflows act on structured output (JSON against a schema) with an explicit "hold" option, not on grep of free text. (§1.9)
- **C11 · State lives in chat.** [ours] *Check:* can a fresh session find the current task, owner and next step from the repo alone (issues, PRs, handoff doc)? (§1.6)
- **C12 · AI steps holding write credentials.** [ours] Some agent actions set up git with whatever token they're given, so the AI could push. *Check:* AI jobs have read-only permissions and `persist-credentials: false`; writes happen in a separate job with no AI. (§1.16)
- **C13 · Locked to one vendor's output feature.** [ours] *Check:* results go through a neutral contract (an output file checked against a schema) with a fail-safe. Schema keywords are portable: some strict modes reject `maxLength` and similar. (§1.17, §1.18)

### D. GitHub Actions security

- **D1 · Script injection.** [field] `${{ github.event.* }}` (titles, bodies, branch names) pasted into `run:` becomes code before the shell sees it. *Check:* grep `run:` blocks for `${{ github.event`, `${{ inputs.` and `head_ref`. *Fix:* pass them through `env:` and quote them.
- **D2 · Privileged triggers plus untrusted code.** [field] `pull_request_target`, `workflow_run` and `issue_comment` run with repo credentials; checking out and running PR code there is the classic "pwn request." *Check:* `grep -rn "pull_request_target\|workflow_run\|issue_comment" .github/workflows`.
- **D3 · Unpinned third-party actions.** [field] Tags can be moved; a full commit SHA can't. *Check:* every non-GitHub-owned `uses:` is pinned to a 40-character SHA with the version in a comment.
- **D4 · Over-broad `GITHUB_TOKEN`.** [field/ours] *Check:* a top-level `permissions:` block on every workflow, default `contents: read`, and each `write` justified in a comment. (§3.6)
- **D5 · Secrets exposure.** [field] *Check:* secrets aren't echoed, passed as command-line arguments or written to artifacts. Prefer OIDC over long-lived keys. Pass secrets only to the steps that need them.
- **D6 · `GITHUB_ENV` / `GITHUB_OUTPUT` injection.** [field] Writing untrusted text there can set variables for later steps. *Check:* grep for `>> $GITHUB_ENV` or `>> $GITHUB_OUTPUT` fed by event data.
- **D7 · Token-made events don't chain.** [ours] A PR or label created with `GITHUB_TOKEN` won't trigger other `pull_request` workflows (`workflow_dispatch` does work). *Check:* any automation that expects a follow-on workflow to fire.
- **D8 · Bot-started runs silently skipped.** [ours] AI actions often refuse runs started by `github-actions[bot]` (e.g. one workflow dispatching another) unless bots are explicitly allowed. *Check:* every workflow started by another workflow has the vendor's allow-bots setting.
- **D9 · Shell and artifact gotchas.** [ours] Under `bash -e`, a step whose last line is `[ cond ] && cmd` fails when `cond` is false. `upload-artifact` v4+ skips hidden files (e.g. `.crew/`) by default. *Check:* use `if …; then …; fi` for the last line; copy outputs to `$RUNNER_TEMP` or set `include-hidden-files`.

### E. Data and integrations

- **E1 · Partial pagination accepted as complete.** [ours] *Check:* the code proves it reached the last page (a declared total, an end cursor) and fails rather than publishing a partial set.
- **E2 · Failure read as absence.** [ours] A 403, 429 or timeout is not "removed" or "closed." *Check:* trace an error path all the way to the user-facing result.
- **E3 · Reruns create duplicates.** [ours] *Check:* run it twice; the second run changes nothing. Dedupe on a stable key (canonical URL, requisition ID).
- **E4 · Overwriting human-owned fields.** [ours] *Check:* sync code has an explicit list of fields it may write; user notes and decisions are never touched.
- **E5 · Private details headed somewhere public.** [ours] *Check:* before publishing or pushing, confirm the target's visibility (`gh api repos/OWNER/REPO --jq .visibility`) and scan for names, personal context, private repo links, secret names and PII.

### F. Docs and status

- **F1 · Plans described as built.** [ours] *Check:* every "verified" claim has a run or output, and "proposed" items are labeled as proposed. (§1.15)
- **F2 · Dated evidence presented as current.** [ours] *Check:* files with dates or old counts are archived or marked frozen, with a pointer to where current data lives. (§2.2)
- **F3 · Outstanding actions scattered.** [ours] *Check:* one checklist answers "what's next, and who owns it?" (§2.4)
- **F4 · Decisions recorded before they're accepted.** [ours] *Check:* the ADR or decision log contains only approved items. (§2.5)

---

## Report format

```markdown
## Review: <project> @ <short SHA> (<date>)

**Verdict:** one line, e.g. "Sound foundation; two blockers in CI security."

### Checks run
| Command / run | Result |
| --- | --- |

### Findings (most severe first)
| ID | Severity | Where | Finding | Evidence | Fix |
| --- | --- | --- | --- | --- | --- |
Severity: blocker · should-fix · nit · pre-existing

### What's working (keep doing)
### Not checked (and why)
### Suggested next step (one)
```

---

## Maintaining this guide
- Add an item when a review finds something new and real. Tag it **[ours]** and log it in `LOG.md`.
- Remove or rewrite an item when it stops being true (tools change; GitHub hardened `pull_request_target` in 2025–26, for example).
- Keep it scannable: one line of smell, one of check, one of fix.

## Sources
- Anthropic: Claude Code best practices (verification, CLAUDE.md length, writer/reviewer pattern, common failure patterns): https://code.claude.com/docs/en/best-practices
- Reward hacking in coding agents: *ImpossibleBench* (https://www.lesswrong.com/posts/qJYMbrabcQqCZ7iqm/impossiblebench-measuring-reward-hacking-in-llm-coding-1) and *EvilGenie* (https://arxiv.org/html/2511.21654v1)
- Slopsquatting: Socket (https://socket.dev/blog/slopsquatting-how-ai-hallucinations-are-fueling-a-new-class-of-supply-chain-attacks), Trend Micro (https://www.trendmicro.com/vinfo/tr/security/news/cybercrime-and-digital-threats/slopsquatting-when-ai-agents-hallucinate-malicious-packages)
- Lethal trifecta: Simon Willison (https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/)
- OWASP Top 10 for Agentic Applications 2026: https://genai.owasp.org/resource/owasp-top-10-for-agentic-applications-for-2026/
- GitHub Actions security: GitHub docs (https://docs.github.com/en/enterprise-cloud@latest/actions/reference/security/secure-use), Wiz "Pwn Request Reckoning" (https://www.wiz.io/blog/github-actions-security-pwn-request-reckoning), Datadog (https://securitylabs.datadoghq.com/articles/case-for-github-actions-security/)
- AGENTS.md open format: https://github.com/coder/agents.md (see also agents.md)
