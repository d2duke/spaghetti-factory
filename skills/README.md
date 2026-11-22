# Skills

Reusable instructions for AI tools. Each skill has its own folder:

| File | Use it with |
| --- | --- |
| `SKILL.md` | Claude (Agent Skills format: frontmatter with `name` and `description`, then instructions). Source of truth. |
| `PORTABLE.md` | Any other AI tool: a paste-ready block plus where to put it. |

| Skill | What it does |
| --- | --- |
| [`project-review`](project-review/) | Loads [`REVIEW-GUIDE.md`](../REVIEW-GUIDE.md) before reviewing a project, PR or agent setup, follows its procedure, and reports in its format. |

When a skill changes, update both files in the same commit.
