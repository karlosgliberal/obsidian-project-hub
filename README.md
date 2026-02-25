# Obsidian Project Hub

Vault template + Claude Code plugin for managing cross-project knowledge with Obsidian.

## What's included

- **Vault template** — Directory structure, Obsidian templates, `.base` views, dashboard, and Blue Topaz theme
- **vault-manager plugin** — Claude Code plugin with commands, skills, agents, and hooks for vault management
- **Kepano obsidian-skills** — Official Obsidian skills for Claude Code (MIT, by Steph Ango)

## Quick start

### 1. Clone and open as vault

```bash
git clone <repo-url> my-knowledge-vault
```

Open the cloned directory as a vault in Obsidian. The Blue Topaz theme and core plugins are pre-configured.

### 2. Install the vault-manager plugin

From any Claude Code session:

```
/plugin install vault-manager@<owner>/obsidian-project-hub
```

### 3. Connect a project

From a project directory in Claude Code:

```
/vault-manager:vault-init
```

This will:
- Ask for your vault path (first time only)
- Analyze the project
- Create a project entry in the vault
- Set up sync rules in the project

### 4. Sync at end of session

```
/vault-manager:vault-sync
```

Updates the vault with session notes, task status changes, and captured knowledge.

## Vault structure

```
├── proyectos/          # One entry per connected project
├── conocimiento/
│   ├── patrones/       # Reusable patterns across projects
│   ├── decisiones/     # Architecture Decision Records (ADRs)
│   └── aprendizajes/   # Debugging insights, learnings
├── tareas/             # Tasks (.md) + dynamic views (.base)
│   └── epicas/         # Epics
├── exploraciones/      # Research and investigations
├── sesiones/           # Session logs
├── plantillas/         # 6 Obsidian templates
└── dashboard.md        # Central navigation hub
```

## Commands

| Command | Description |
|---------|-------------|
| `/vault-manager:vault-init` | Connect current project to the vault |
| `/vault-manager:vault-sync` | Sync current session to the vault |

## Skills

- **vault-knowledge** — Auto-triggered when you mention vault, patterns, tasks, or knowledge management

## Conventions

- All notes use YAML frontmatter with `tags`, `date`, and `status`
- Files named in kebab-case
- Cross-references use `[[wikilinks]]`
- `.base` files are dynamic views (Kanban, tables) — they filter existing notes

See `CLAUDE.md` for full conventions.

## License

MIT
