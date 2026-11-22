---
name: "project-review"
description: "Use when asked to review a code project, repo, pull request, or AI agent setup; loads Colin's spaghetti-factory REVIEW-GUIDE.md first and reports in its format."
---

# Project review

Before looking at the project, load the review guide and follow it.

## 1. Load the guide (always first)
Read the current version. Don't work from memory.
- With the GitHub CLI: `gh api repos/d2duke/spaghetti-factory/contents/REVIEW-GUIDE.md -H "Accept: application/vnd.github.raw"`
- Otherwise, fetch: https://github.com/d2duke/spaghetti-factory/blob/main/REVIEW-GUIDE.md

Also skim the top entry of `LOG.md` in the same repo for anything recent.

## 2. Run the review
- Follow the guide's ground rules and its 7-step procedure in order.
- Use the catalog (A to F) as the checklist, and cite findings by ID (e.g. B1, D3).
- Read-only unless the owner asks for changes. List every destructive action and get a yes first.
- Use full git history, not a shallow clone, before comparing branches.
- Try to disprove each finding before reporting it. Report only correctness, security, data-loss and missed-requirement gaps.

## 3. Report
Use the guide's report format: verdict, checks run, findings table (ID, severity, where, finding, evidence, fix), what's working, what wasn't checked, and one suggested next step. Visuals help: add a simple diagram when structure or flow matters.

## 4. Feed the notebook
If the review found a pitfall the catalog doesn't cover, offer to add it to spaghetti-factory: a new [ours] item in REVIEW-GUIDE.md plus a dated LOG.md entry. spaghetti-factory is a PUBLIC repo. Strip private project names, private repo links, personal context and secret names before pushing.