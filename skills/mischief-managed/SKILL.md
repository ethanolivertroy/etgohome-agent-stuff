---
name: mischief-managed
description: Use when finishing a Pi, Codex, Cursor, Claude, or other coding-agent session and wanting to save a lean Markdown session note into your Obsidian vault with decisions, code changes, validation, caveats, and follow-ups.
---

# Mischief Managed - Session Summary to your Obsidian vault

Write a compact session note into your Obsidian vault so the work is searchable and resumable later. This is a summary, not a full transcript.

## Choosing the Agent Folder

Notes are organized by coding agent. The agent folder is `{Agent}`.

`{Agent}` is the coding agent running this session. Detect it from the current runtime/client context, not from where this skill file is installed. Common values: `Claude`, `Codex`, `Cursor`, `Pi`, `OpenCode`.

Resolve `{Agent}` in this priority order:

1. If the user names or corrects the agent in the current request, use that value.
2. If the active client/runtime context identifies the agent, use it:
   - Cursor Desktop, Cursor CLI, Cursor Agent, or `.cursor` session context -> `Cursor`
   - Claude Code or Claude CLI/session context -> `Claude`
   - Codex CLI/session context -> `Codex`
   - Pi CLI/session context -> `Pi`
   - opencode CLI/session context -> `OpenCode`
3. If environment or process context clearly identifies the running CLI, use that signal.
4. If still ambiguous, ask once before writing the note.

Do not assume the agent based on where this skill file is installed.

## Obsidian Vault Location

Within the vault, write to:

```text
{vault}/Agent-Sessions/{Agent}/
```

Resolve `{vault}` in this priority order:

1. If the user gives a vault path or note path, use it.
2. If `OBSIDIAN_VAULT_PATH` is set, use it.
3. If `~/Obsidian` exists, use it.
4. Otherwise ask once for the Obsidian vault path before writing.

So the default target is `~/Obsidian/Agent-Sessions/{Agent}/`.

Do not silently write outside an Obsidian vault unless the user explicitly asks.

If the user keeps notes somewhere else (Notion, Logseq, a plain Markdown folder), write the same note and keep the Marauder's Map there instead, following that tool's conventions. Use whatever tooling is available for it, such as a Notion MCP server, and fall back to a local Markdown file if there is none.

Filename:

```text
{YYYY-MM-DD-HHMM}-{slug}.md
```

Use a 2-4 word kebab-case slug based on the completed work.

Prefer Obsidian-friendly Markdown: wikilinks are fine when the target note is obvious, but do not invent a vault taxonomy.

## The Marauder's Map

Maintain a single index note that reveals every session, like the Marauder's Map reveals every footstep in the castle.

Location:

```text
{vault}/Agent-Sessions/Marauders-Map.md
```

Shape:

```markdown
# The Marauder's Map

*"I solemnly swear that I am up to no good."*

Messrs Moony, Wormtail, Padfoot, and Prongs present every footstep taken across the castle.

## {Agent}
- [[{YYYY-MM-DD-HHMM}-{slug}]] - {PRETTY_DATE} - one-line summary

*"Mischief managed."*
```

Each session is one footstep: an Obsidian wikilink to the note, its date, and a one-line summary. Newest first within each `## {Agent}` section.

Update the map in two ways:

1. **On save (do this every time):** after writing a session note, add its line under the matching `## {Agent}` heading (create the map file, header, footer, or the heading if missing). If a line for that note already exists, update it in place rather than duplicating.
2. **On request (full rebuild):** when the user asks to redraw or rebuild the Marauder's Map, scan every `{vault}/Agent-Sessions/{Agent}/*.md` note and regenerate the whole map from scratch, preserving the incantation header and footer.

## Metadata

Run one shell command to collect context:

```bash
git rev-parse --abbrev-ref HEAD 2>/dev/null || echo ""; basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null || echo ""; date +%Y-%m-%d-%H%M; date +"%b %d, %Y"; pwd; git remote get-url origin 2>/dev/null || echo ""
```

Map output to:

- `BRANCH`
- `REPO`
- `DATETIME`
- `PRETTY_DATE`
- `DIR`
- `REPO_URL`

If the git values are blank, omit them from frontmatter.

## Note Shape

```markdown
---
date: {DATETIME}
branch: {BRANCH}
repo: {REPO}
directory: {DIR}
tags: [agent-session, {agent-lowercase}-session, {tag1}, {tag2}]
---

# Session: {SLUG} - {PRETTY_DATE}

## Summary
2-3 sentences describing what changed and why.

## What Was Discussed
- Short bullets with useful context.

## Key Decisions
- decision - rationale

## Code Changes
- `path` - what changed and why

## Tests & Validation
- `command` - result

## Risks / Caveats
- Anything unresolved, unverified, or worth revisiting.

## Questions & Follow-ups
- [ ] Concrete next action
```

Required sections: `Summary`, `What Was Discussed`, `Key Decisions`, and `Questions & Follow-ups`.

Include `Code Changes`, `Tests & Validation`, and `Risks / Caveats` only when meaningful.

## Content Rules

- Keep it lean. Capture durable facts, not every exchange.
- Preserve exact commands, filenames, issue IDs, URLs, and error messages when they matter.
- Do not include secrets, tokens, cookies, API keys, or private credential material. Redact as `{REDACTED}` if needed.
- Do not post to external trackers or services unless the user explicitly asks in the current session.

## Confirmation

After writing the note, update The Marauder's Map (see the location section above) with this session's footstep, then respond with:

- the note location (absolute path or URL)
- one-line summary of what was captured

End the reply with: `Mischief managed.`
