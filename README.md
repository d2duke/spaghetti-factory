# 🍝 Spaghetti Factory

A shared engineering learning library: a running notebook of what works (and what doesn't) when building software with AI coding agents.

## Shared use across projects

Use this library as advisory engineering guidance for planning, implementation and independent review, regardless of project or AI engine. Project operating instructions, security boundaries, approved task scope and human approval gates take precedence. Automated consumers should use an explicitly reviewed, versioned snapshot rather than treating mutable external text as instructions. Record the revision used when practical.

Feed back reusable, evidence-backed lessons through reviewed changes to this library. This is a public repository: generalize private work, omit identifying details, private links, credentials and raw logs. Never automatically publish project data or let the library authorize new access, actions or spending.

**Reviewing a project? Start with [`REVIEW-GUIDE.md`](REVIEW-GUIDE.md).**

| File | What it holds | How it changes |
| --- | --- | --- |
| [`REVIEW-GUIDE.md`](REVIEW-GUIDE.md) | What a reviewer (agent or person) reads before looking at any project: a procedure, a catalog of common pitfalls with checks, and a report format. | Add an item when a review finds something new and real; remove items that stop being true. |
| [`BEST-PRACTICES.md`](BEST-PRACTICES.md) | The current playbook, organized by topic. Each practice says what to do, why, and how to check it. | Edited in place. When a practice changes, rewrite it and note why in the log. |
| [`LOG.md`](LOG.md) | Dated entries: what happened in a session, what was learned, links to the evidence. | Append only. Newest entry at the top. |
| [`skills/`](skills/) | Reusable instructions for AI tools, e.g. [`project-review`](skills/project-review/), which makes any review load the guide first. Each comes as `SKILL.md` (Claude) and `PORTABLE.md` (paste into any other tool). | Update both files together. |

## Adding to it
1. Add a dated entry to the top of `LOG.md`: context, what happened, lessons, links.
2. If a lesson is a durable rule, add it to `BEST-PRACTICES.md` (or improve an existing one) and tag it with the log date it came from.
3. Give a practice a "how to check" line only if you can actually check it.

Every lesson should have earned its place in real work. No copying advice that hasn't been tried.
