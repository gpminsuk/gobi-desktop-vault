---
description:
globs:
alwaysApply: true
---

# important-instruction-reminders
Do what has been asked; nothing more, nothing less.
NEVER create files unless they're absolutely necessary for achieving your goal.
ALWAYS prefer editing an existing file to creating a new one.
NEVER proactively create documentation or README files.

# Generic AI Agent Rules

*These rules apply to all AI agents (Claude Code, Gemini, Codex) working in this Gobi Desktop vault.*

## Core Mission & Principles
- **Your mission is to enhance and organize user's knowledge**
	- Don't add your internal knowledge unless explicitly asked to do so
- Most commands are based on existing prompts and workflows (locations below)
	- But note that default settings (e.g. input/output) can be overridden for each run
- You're expected to run autonomously for most prompts & workflow runs
	- Use your judgment to complete the task unless asked otherwise

## Prompts & Workflows
- Orchestrator config in `orchestrator.yaml` (root) — defines I/O routing, schedules, triggers
- Prompts can be found in `_Settings_/Prompts`
  - Naming: `Full Title (ABC).md` (3-letter abbreviation in parentheses)
  - Frontmatter: `title / abbreviation / category / created` — see `_Settings_/Templates/Prompt Template.md`
  - I/O paths live in `orchestrator.yaml`, NOT in the prompt frontmatter
- Agent system prompts in `_Settings_/Agents` (e.g. Real-time Voice Assistant)
- Skills can be found in `.claude/skills`
- Templates (of md docs) in `_Settings_/Templates`
- Knowledge Tasks in `_Settings_/Tasks` (only when requested)
- Cross-cutting know-how (cookbooks, trial-and-error logs) in `_Settings_/Notes/` — read BEFORE related work, append AFTER discoveries
- Each command can be called using abbreviations
- Check this first for new command (especially if it's abbreviations)

## Cross-Framework Awareness
- This vault is **interoperable** with the [cmds-system-files](https://github.com/johnfkoo951/cmds-system-files) framework by convention, not by merger. The two frameworks have different frontmatter schemas, folder taxonomies, and wikilink rules.
- When you encounter content authored under cmds conventions (numeric folders `100`–`900`, `date created` / `date modified` frontmatter, emoji-prefixed wikilinks, English `description` field), treat it as **valid-but-foreign**. Do not rewrite it to Gobi Desktop vault conventions unless the user asks.
- The authoritative rules for THIS vault remain in `CLAUDE.md` (this file).

## Skills
- Project skills are located in `.claude/skills/`
- Each skill folder contains a `SKILL.md` with instructions
- To use a skill, read the corresponding `SKILL.md` file first
- Available project skills:
  - `gobi-onboarding` - Gobi Desktop voice onboarding flow
  - `create-gobi-homepage` - generate a Brain homepage (home.html) from 4 style templates via an interactive interview (legacy `/CBH` still resolves)
  - `writeup` - drafts 2-3 writeup options from the current conversation, presents via AskUserQuestion, then saves to `Articles/` and/or posts to gobi space (`/writeup` for approval mode, `/writeup bypass` for Recommended-defaults auto-run)
- Gobi CLI is exposed as user-invocable harness skills (no local folder needed):
  - `gobi:gobi-core` - auth, vault init, space warp, CLI updates, session management
  - `gobi:gobi-space` - posts and replies in community space (`gobi space`) and global feed (`gobi global`)
  - `gobi:gobi-saved` - personal saved notes and bookmarked posts
  - `gobi:gobi-sense` - activity and transcription records
  - `gobi:gobi-draft` - agent-authored drafts (list, prioritize, action, revise)
  - `gobi:gobi-vault` - publish/unpublish vault profile (root `PUBLISH.md`), sync local ↔ webdrive
  - `gobi:gobi-media` - image/video/avatar generation
  - `gobi:gobi-homepage` - build/edit vault homepages

## Content Folders (Data Flow)

The vault's content flow has two stages, each in a dedicated root folder:

```
Ingest/         → Articles/
(raw external)   (user thinking)
```

- **`Ingest/`** — raw data collected from outside sources
  - `Ingest/Clippings/` — web clippings, transcripts, captures (CAE enriches these)
  - `Ingest/Research/` — research briefings and reference material (DRB writes here)
  - `Ingest/Chats/` — chat session history (orchestrator `chat_history_dir`)
  - `Ingest/Captures/` — ambient canvas / voice captures (orchestrator `captures_dir`)
- **`Articles/`** — user-authored content where the user infuses their own thinking with ingested data. Authored by the user, often through dialogue with a deep-thinking agent.
- **`Context/`** — user-context reference data: events (conferences, meetups, talks), interests, preferences, profile-adjacent metadata. Read/written by onboarding, research, and conversational agents.
  - `Context/interest.md` — user's interest list (consulted by DRB, CAE TOPIC mode, DTA seed selection; written/updated by onboarding)
  - `Context/preference.md` — user's output/style/cadence/source preferences (consulted by DRB for briefing params, CAE for output formatting + source filtering, DTA for dialogue length, RVA for response style; written by onboarding + explicit user "다음부터는 ~" requests)
  - `Context/YYYY-MM-DD Event Name/` — one folder per event with `event.md` + flat session files (no `Sessions/` subfolder)
- **`AI/`** — agent workspace for machine-generated intermediate outputs (gitignored, not part of the user-visible flow).

## Search over files
- For searching over topic or dates, start from `Articles` folder
- Follow markdown link to find related files (use `find` to find exact location)
* **Consider `.gitignore` when searching files**: When finding file lists or searching content, use `respect_git_ignore=False` option to include all relevant files that might otherwise be excluded by `.gitignore`.

## 📝 Content Creation Requirements
### General Guidelines
- **Include original quotes** in blockquote format
- **Add detailed analysis** explaining significance
- Structure by themes with clear categories
- **Use wiki links with full filenames**: `[[YYYY-MM-DD Filename]]`
- **Tags use plain text in YAML frontmatter**: `tag` not `#tag` in YAML
  - Example:
```yaml
tags:
  - journal
  - daily
```

### Writing Style
- **Tight layout**: Do not use horizontal dividers (`---`) between sections
- **Paragraph cohesion**: Write related sentences as a single paragraph (minimum 2-3 sentences)
  - Avoid paragraphs with only one sentence standing alone
  - Combine short sentences logically into one

### Markdown Table Formatting
- **Blank line required before tables**: Markdown tables must have a blank line immediately before them to render properly

### Diagram Standards
- **Write diagrams in Mermaid**: Use Mermaid instead of ASCII art

### Table vs Diagram Selection
- **Use tables for**: Attribute-value mappings, comparisons, option listings (structured data)
- **Use Mermaid for**: Flows, processes, relationships, time sequences (visual flows)
- **Optimize document length**: Choose the format that expresses the same information more compactly

### Link Format Standards
- Use Link Format below for page properties:
```yaml
  - "[[Page Title]]"
```
- For files in AI folder, omit "AI/" prefix for brevity
- Example: `[[Roundup/2025-08-03 - Claude Code]]` not `[[AI/Roundup/2025-08-03 - Claude Code]]`

### Embed Format Standards
- **Embed (transclude) with `![[...]]`**: prefix the wikilink with `!` to embed content inline rather than just link to it
- **Images**: `![[relative/path/to/image.png]]` — add `|width` for sizing: `![[image.png|320]]`
- **Videos**: `![[relative/path/to/clip.mp4]]` — same syntax
- **Markdown file embed**: `![[file.md]]` to embed full file, `![[file.md#Section]]` to embed a specific section
- Use vault-relative paths (no leading `/`). Standard markdown image syntax `![alt](url)` is only for external URLs

### 📁 Output File Management
- Create analysis files in `AI/*/` folder unless instructed otherwise
- Naming: `YYYY-MM-DD [Project Name] by [Agent Name].md`
- Include source attribution for every insight

### Inline Links for Research Documents
- **Insert related links throughout the body of research/analysis documents**
- Add contextual links where relevant content is mentioned, not just in the References section
- **Link format**:
  - `→ **Deep analysis**: [[path/to/file|display text]]`
  - `→ **Related research**: [[path/to/file#section-name|display text]]`

### Properties & Frontmatter Standards
- Use a single YAML block at top (`---` … `---`). Leave one blank line after it.
- Keys are lowercase and consistent: `title`, `source` (URL), `author` (list), `created` (YYYY-MM-DD HH:MM:SS), `tags` (list)
- **created property includes actual creation time**: When AI generates a document, record both date and time
- Avoid duplicates like `date` vs `created`
- Tags are plain text (no `#`) and indented list; authors may be wiki links wrapped in quotes
- Quote values that contain colons, hashes, or look numeric to avoid YAML casting
- After frontmatter, start with a section heading — no loose text or embeds before the first heading

## 🔄 Additional Principles

### Update over duplicated creation
- If a file already exists for that date, update it (do not create a new one)
  - When updating, don't just append new content; revise with overall consistency in mind (duplication is a sin)

### Language Preferences
- Use the `primaryLanguage` from `.gobi/settings.yaml` as the default language for all output (English is fine, say, to quote original note)
- For voice/conversation: match the user's spoken language; fall back to `primaryLanguage` if ambiguous

### User Clarification — Structured Choice over Open Questions

When you need user input on a decision with **2-4 discrete options** that meaningfully change your next action, present them as a **structured choice**, not an open question. Reduces back-and-forth, surfaces options the user might not know exist, and produces a cleaner decision trail.

**Tool / rendering by environment**:
- **Claude Code / chat**: use the `AskUserQuestion` tool — renders as clickable options with descriptions. Mark the best pick with `(Recommended)` and put it first.
- **Voice (RVA, DTA via voice)**: verbalize as numbered or labeled options.
  > "두 가지 방법이 있어요. 첫째, 새 글로 쓰기. 둘째, 기존 글에 이어 붙이기. 어느 쪽이 끌리세요?"

**When to use**:
- ✅ Options are mutually exclusive and the choice branches your next step
- ✅ User might not realize all options exist
- ✅ Multiple candidate files/sources/titles found and you need to pick one

**When NOT to use**:
- ❌ Simple yes/no — just ask "할까요?"
- ❌ Open exploration is the goal (e.g. DTA SURFACE step — leave the question open)
- ❌ 5+ options — narrow down first, then offer choice
- ❌ The "right" answer is obvious from context — just do it

### 🔗 Critical: Wiki Links Must Be Valid
- **All wiki links must point to existing files**
- Use complete filename: `[[2025-04-09 세컨드 브레인]]` not `[[세컨드 브레인]]`
  - Add section links when possible (using `#` suffix)
- **Section-level links required when citing sources**
  - When quoting or referencing content from other documents, always link to the specific section
  - Example: `[[2025-12-01 Meeting Notes#2. Discussion Items|Meeting Notes]]`
- Verify file existence before linking
  - Fix broken links immediately
- **Link to original sources, not aggregations**
  - Always link to the original article, clipping, or document where content first appeared
  - Example: Link to `[[Ingest/Clippings/2025-08-15 역스킬 현상]]` directly, not via an intermediate index file
  - This maintains proper source attribution and traceability

## Source/Prompt-specific Guidelines
### Limitless Link Format
- **Correct path**: `[[Limitless/YYYY-MM-DD#section]]` (no Ingest prefix)
- **Always verify section exists**: Check exact header text in source file
- **Section headers are usually Korean**: Match them exactly as written
- **If unsure about section**: Link to file only `[[Limitless/YYYY-MM-DD]]`

### Heading Structure Guidelines
- Clippings (CAE/ICT): begin with `## Summary`, then `## Improve Capture & Transcript (ICT)`, then transcript
- ICT means improve the transcript (correct grammar, translate to Korean, structure with h3), not summarize. Keep length comparable to source; summaries live only under `## Summary`
- Lifelog: use H1 `# YYYY-MM-DD Lifelog - <Assistant>` then H2 sections (Monologues, Conversations, etc.)
- Articles: start with H2 summary; avoid duplicating title as H1

## Quality Standards
- Validate all wiki links resolve to existing files/sections; fix broken links immediately
- Focus on meaningful content over metadata files
- Don't ask permission for any non-file-changing operations (search/list/echo etc)
- Always use local time (usually in Seattle area) for processing requests

## Gobi CLI Features
- Gobi CLI는 harness 레벨의 user-invocable 스킬(`gobi:*`)로 노출됨
- **Space / Global Feed**: `gobi:gobi-space` — 커뮤니티 스페이스(`gobi space`)와 글로벌 피드(`gobi global`)의 posts/replies 읽기·쓰기
- **Saved**: `gobi:gobi-saved` — 개인 saved notes 및 북마크한 posts 관리
- **Sense**: `gobi:gobi-sense` — activity 및 transcription 기록 조회
- **Draft**: `gobi:gobi-draft` — 에이전트가 작성한 draft 관리 (list/prioritize/action/revise)
- **Vault**: `gobi:gobi-vault` — vault 프로필 publish/unpublish (root `PUBLISH.md`), local ↔ webdrive 동기화
- **Core**: `gobi:gobi-core` — auth, vault init, space warp, CLI 업데이트, session 관리

---

# Claude Code Specific Rules

## 📋 Task Management
### TodoWrite Usage
- **Always use TodoWrite** for multi-step projects (3+ steps)
- Mark ONE task `in_progress` at a time
- Mark `completed` immediately after finishing

## Version Control
### Commit Message Format for Workflows
- Use format: `Workflow: [Name] - YYYY-MM-DD`
- Only include affected files (don't commit unaffected files)
- Include brief summary of changes
- Add emoji and Co-Authored-By signature
- Example:
```
Workflow: DIR - 2025-08-28

Daily Ingestion and Roundup:
- Processed lifelog from Limitless
- Updated daily roundup
- Added topic knowledge updates

🤖 Generated with [Claude Code](https://claude.ai/code)

Co-Authored-By: Claude <noreply@anthropic.com>
```

## Claude Code Tool Usage
### Task Tool Priority
- **Use Task tool** for comprehensive searches and "find all X" requests
- Leverage specialized agents when available

### 🔍 Search Strategy
- Use comprehensive search tools for "find all X" requests
- Use multiple languages (한글 / English) for max recall
- **Read multiple files in parallel** for efficiency

## Continuous Improvement Loop
### Find rooms for improvement
- By evaluating output based on prompt
- By using user feedback

### Suggest ways
- Improvement to existing prompts
- New or revised workflows

## Additional Guidelines
### Workflow Completion
- Run all steps (i.e. prompts) are run when running a workflow
	- Keep input/output requirements (file path/naming)
- Ensure all workflow steps are completed

### Parallelization Opportunities
- 파일 고치기/찾기는 대부분 병렬화가 가능
- 병렬화를 통해 시간 단축할 수 있는 기회를 찾고 수행

### Data Source Preferences
- Don't use git status for checking update; read actual files from folder
