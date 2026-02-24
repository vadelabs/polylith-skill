# polylith-skill

A Claude Code skill for working in [Polylith](https://polylith.gitbook.io/polylith/) monorepos.

Gives Claude Code accurate knowledge of:
- Polylith workspace structure (components, bases, projects)
- Interface namespace conventions
- Dependency rules between bricks
- Common `poly` CLI commands
- Cross-platform `.cljc` patterns
- Stuart Sierra Clojure naming conventions

## Install

### Option 1: Into your project (team-shared)

```bash
cp -r .claude/skills/polylith YOUR_PROJECT/.claude/skills/
```

Then commit `.claude/skills/` so every teammate gets it.

### Option 2: Personal skill (available across all your projects)

```bash
cp -r .claude/skills/polylith ~/.claude/skills/
```

## Usage

Once installed, tell Claude Code to load it when working on Polylith code:

```
/polylith
```

Or reference it in your `CLAUDE.md`:

```markdown
Load the polylith skill when working in this repository.
```

## What's inside

| File | Contents |
|------|----------|
| `SKILL.md` | Core architecture rules and workspace structure |
| `conventions.md` | Clojure naming conventions (Stuart Sierra style) |
| `commands.md` | `poly` CLI and common workflow commands |
| `patterns.md` | Interface pattern, component creation, `.cljc` usage |

## License

MIT
