# Mischief Managed

A portable [Agent Skill](https://agentskills.io) that saves a lean, searchable session note at the end of a coding-agent session: the decisions you made, the code you changed, what you validated, open risks, and follow-ups.

"Mischief managed." Close out a session and leave a trail you (or your next session) can pick up later.

It works across coding agents and lets you keep your notes in whatever knowledge base you already use.

## Install

This skill is part of [etgohome-agent-stuff](../../README.md#install). To use it with an
[Agent Skills](https://agentskills.io)-compatible runtime, copy or symlink this folder
(`skills/mischief-managed/`) into the agent's skills directory:

| Agent    | Install location                          |
| -------- | ----------------------------------------- |
| Claude   | `~/.claude/skills/mischief-managed/`      |
| Cursor   | `~/.cursor/skills/mischief-managed/`      |
| Codex    | `~/.codex/skills/mischief-managed/`       |
| Pi       | `~/.pi/agent/skills/mischief-managed/`    |
| opencode | `~/.config/opencode/skills/mischief-managed/` |

## Where notes go

Obsidian by default: notes go under `Agent-Sessions/<Agent>/` in the vault at
`$OBSIDIAN_VAULT_PATH` or `~/Obsidian`. Keep notes somewhere else (Notion, Logseq, a plain
Markdown folder)? Tell your agent where, and it writes the same note there instead.

## The Marauder's Map

Whatever knowledge base you pick, the skill also maintains a single index note - `Marauders-Map.md` at the root of your `Agent-Sessions/` area (a page titled "The Marauder's Map" for Notion). Like the real thing, it reveals every footstep: a link to each session, grouped by agent, newest first, wrapped in the incantations.

It is kept current two ways:

- **On save** - each time the skill writes a session note, it adds that note's footstep to the map.
- **On request** - ask your agent to "redraw the Marauder's Map" and it rescans every session note and rebuilds the whole map.

## License

MIT - see [LICENSE](../../LICENSE).
