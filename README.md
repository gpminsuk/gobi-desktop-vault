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
│   ├── Agents/              # Agent system prompts (e.g. RVA)
│   ├── Prompts/             # Workflow/batch agent prompts
│   ├── Templates/           # Markdown templates
│   └── Logs/                # Execution logs
├── Ingest/                  # Raw data from outside sources
│   ├── Clippings/           # Web clippings, transcripts, captures
│   └── Research/            # Research briefings, reference material
├── Articles/                # User-authored content (user's thinking on ingested data)
├── Topics/                  # Topical index / LLM wiki
├── orchestrator.yaml        # Agent orchestration config
├── AGENTS.md                # Generic AI agent rules
├── CLAUDE.md                # Claude-specific rules
├── PUBLISH.md               # Vault profile (published via gobi-vault)
└── README.md
```

## Content Flow

```
Ingest/  →  Articles/  →  Topics/
(raw)       (user)        (wiki)
```

- **Ingest/** — collected from outside (clippings, research). Agents may enrich items in place.
- **Articles/** — where you write. User-authored notes that synthesize ingested data with your own thinking.
- **Topics/** — derived topical index maintained by agents (TIU). Search starts here.

User content folders track only the folder skeleton (`.gitkeep`) in git; the contents themselves are gitignored. `AI/` is an agent workspace and is fully gitignored.

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
