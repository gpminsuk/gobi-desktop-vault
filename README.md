# Gobi Desktop Vault

An Obsidian vault spec for the Gobi Desktop AI harness system. Forked from [AI4PKM](https://github.com/jykim/AI4PKM) and adapted for Gobi.

## Structure

```
gobi-desktop-vault/
├── .claude/
│   └── skills/              # Claude Code skills
├── .gobi/
│   └── settings.yaml        # Gobi runtime settings (language, mic, LLM provider)
├── _Settings_/              # Vault configuration and system files
│   ├── Agents/              # Agent system prompts
│   ├── Prompts/             # AI agent prompts
│   ├── Templates/           # Markdown templates
│   └── Logs/                # Execution logs
├── orchestrator.yaml        # Agent orchestration config
├── AGENTS.md                # Generic AI agent rules
├── CLAUDE.md                # Claude-specific rules
├── GEMINI.md                # Gemini-specific rules
└── VAULTS.md                # Multi-vault registry
```

User content folders (`AI/`, `Ingest/`, `Journal/`, `Topics/`, `_Settings_/Tasks/`) are gitignored — created on first use.

## Usage

1. Clone this repository
2. Open the folder as an Obsidian vault
3. Run the Gobi CLI pointed at this vault path

## Configuration

- `orchestrator.yaml` — agent routing, polling, and capture settings
- `.gobi/settings.yaml` — voice/language preferences and LLM provider

## Agent Rules

Agent behavior is governed by three files at the vault root:

- `AGENTS.md` — generic rules for all AI agents (Claude, Gemini, Codex)
- `CLAUDE.md` — Claude Code specific overrides
- `GEMINI.md` — Gemini CLI specific overrides

## Related Projects

- [AI4PKM](https://github.com/jykim/AI4PKM) — upstream PKM framework this vault was forked from
- [claude-obsidian-skills](https://github.com/jykim/claude-obsidian-skills) — reusable Claude skills
