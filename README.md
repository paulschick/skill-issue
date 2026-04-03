# Skill Issue

AI agent skills for version-constrained development. Skills teach AI assistants to write correct code for specific
language versions, preventing use of features that aren't available in the target runtime.

## Skills

| Skill                        | Description                                                  | Trigger                  |
|------------------------------|--------------------------------------------------------------|--------------------------|
| [java-8](skills/java/java-8) | Java 8 patterns and version constraints for legacy codebases | When writing Java 8 code |

## Installation

Copy the skills you need into your AI agent's skills directory:

| Agent               | Skills Directory                |
|---------------------|---------------------------------|
| **Claude Code**     | `~/.claude/skills/`             |
| **OpenCode**        | `~/.config/opencode/skills/`    |
| **Gemini CLI**      | `~/.gemini/skills/`             |
| **Cursor**          | `~/.cursor/skills/`             |
| **VS Code Copilot** | `~/.copilot/skills/`            |
| **Codex**           | `~/.codex/skills/`              |
| **Windsurf**        | `~/.codeium/windsurf/skills/`   |
| **Antigravity**     | `~/.gemini/antigravity/skills/` |

### Quick Start

```bash
# Clone the repository
git clone https://github.com/paulschick/skill-issue.git

# Copy a skill to your agent's directory (example: Claude Code)
cp -r skill-issue/skills/java/java-8 ~/.claude/skills/

# Or copy all skills
cp -r skill-issue/skills/* ~/.claude/skills/
```

### Skill Structure

Inside each skills directory, every skill lives in its own folder with a `SKILL.md`:

```
skills/
  java/
    java-8/SKILL.md
```

## Contributing

Contributions welcome from humans and AI agents. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

### Adding a Skill

1. Create `skills/<language>/<skill-name>/SKILL.md`
2. Follow the [skill template](CONTRIBUTING.md#skillmd-requirements)
3. Open a pull request

## License

[MIT](LICENSE)
