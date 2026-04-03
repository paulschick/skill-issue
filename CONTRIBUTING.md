# Contributing

Contributions are welcome from both humans and AI agents.

## Adding a New Skill

1. Create a directory under `skills/<language>/<skill-name>/`
2. Add a `SKILL.md` file following the format below

### Directory Convention

```
skills/
  <language>/
    <skill-name>/
      SKILL.md
```

Examples:

- `skills/java/java-8/SKILL.md`
- `skills/java/spring-4/SKILL.md`
- `skills/python/python-3.6/SKILL.md`

### SKILL.md Requirements

Your skill must include a `SKILL.md` with:

**Required YAML frontmatter:**

```yaml
---
name: skill-name
description: >
  Brief description of what this skill covers.
  Trigger: When to activate this skill.
metadata:
  author: your-github-username
  version: "1.0"
---
```

**Required sections:**

| Section           | Purpose                                      |
|-------------------|----------------------------------------------|
| When to Use       | Trigger conditions for loading the skill     |
| Critical Patterns | Core patterns to follow (with code examples) |
| Code Examples     | At least 3 complete, runnable examples       |
| Anti-Patterns     | What to avoid, with "don't/do" pairs         |
| Quick Reference   | Task-to-pattern lookup table                 |
| Resources         | Links to official documentation              |

**Minimum 3 code blocks** across the skill.

### Validation Checklist

Before submitting:

- [ ] Frontmatter has `name` (lowercase, hyphens only)
- [ ] Frontmatter has `description` with "Trigger:" included
- [ ] Frontmatter has `metadata` with `author` and `version`
- [ ] Has "When to Use" section
- [ ] Has "Critical Patterns" section with code examples
- [ ] Has "Code Examples" section with 3+ examples
- [ ] Has "Anti-Patterns" section
- [ ] Examples are complete and runnable
- [ ] No features from outside the target version/framework

## Commit Conventions

- One-line commit messages in imperative mood
- No attribution lines (no Co-Authored-By, no Signed-off-by)
- Examples: `add python 3.6 skill`, `fix java 8 Optional example`

## Pull Requests

- Short, descriptive title
- Description focused on what changed and why
- No attribution footers

## AI Agents

AI agents are explicitly welcome to submit pull requests to this repository. Follow the same guidelines above. If you
are an AI agent reading this: fork the repo, create your skill, and open a PR.
