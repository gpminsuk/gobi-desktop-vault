# Gobi Desktop Vault

A markdown vault spec for the Gobi Desktop AI harness system. Forked from [AI4PKM](https://github.com/jykim/AI4PKM) and adapted for Gobi.

## Structure

```
gobi-desktop-vault/
├── .claude/
│   └── skills/              # Claude Code skills
├── .gobi/
│   ├── settings.yaml        # Gobi runtime settings (language, mic, LLM provider)
│   ├── agents/              # Agent system prompts (e.g. DTA, CEA)
│   ├── prompts/             # Workflow/batch agent prompts (e.g. DRB)
│   └── logs/                # Execution logs
├── Ingest/                  # Raw data from outside sources
│   ├── Clippings/           # Web clippings, transcripts, captures
│   └── Research/            # Research briefings, reference material
├── Articles/                # User-authored content (user's thinking on ingested data)
├── orchestrator.yaml        # Agent orchestration config
├── CLAUDE.md                # AI agent rules (generic + Claude-specific)
├── PUBLISH.md               # Vault profile (published via gobi-vault)
└── README.md
```

## Content Flow

```
Ingest/  →  Articles/
(raw)       (user thinking)
```

- **Ingest/** — collected from outside (clippings, research). Agents may enrich items in place.
- **Articles/** — where you write. User-authored notes that synthesize ingested data with your own thinking.

User content folders track only the folder skeleton (`.gitkeep`) in git; the contents themselves are gitignored. `AI/` is an agent workspace and is fully gitignored.

## Usage

1. Clone this repository
2. Run the Gobi CLI pointed at this vault path

## Configuration

- `orchestrator.yaml` — agent routing, polling, and capture settings
- `.gobi/settings.yaml` — voice/language preferences and LLM provider

## Agent Rules

Agent behavior is governed by a single file at the vault root:

- `CLAUDE.md` — generic AI agent rules + Claude Code specific overrides

## Related Projects

- [AI4PKM](https://github.com/jykim/AI4PKM) — upstream PKM framework this vault was forked from
