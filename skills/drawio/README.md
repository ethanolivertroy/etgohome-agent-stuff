# drawio

A skill for generating native draw.io diagrams as `.drawio` files and optionally exporting them to PNG, SVG, PDF, or JPG.

PNG, SVG, and PDF exports use draw.io Desktop's CLI with embedded diagram XML, so exported files remain editable in draw.io.

## Install

This skill is part of [etgohome-agent-stuff](../../README.md#install). To use it with an
[Agent Skills](https://agentskills.io)-compatible runtime, copy or symlink this folder
(`skills/drawio/`) into the runtime's skills directory. It was previously the standalone
`pi-drawio` package.

## Usage

```text
/skill:drawio create a flowchart for password reset
/skill:drawio png architecture diagram for a web app with CDN, API, queue, workers, and Postgres
/skill:drawio svg ER diagram for users, teams, projects, and tasks
```

You can also ask naturally; the agent should load the skill when your request mentions
diagrams, draw.io, `.drawio`, PNG/SVG/PDF export, architecture diagrams, ER diagrams,
flowcharts, mockups, and related tasks.

## What it does

- Generates native draw.io `mxGraphModel` XML.
- Writes `.drawio` files directly; no server-side conversion required.
- Optionally exports with draw.io Desktop CLI.
- Embeds diagram XML into PNG, SVG, and PDF exports.
- Opens the result when possible.
- Includes layout quality guidance to reduce clipping, label collisions, and messy edge routing.
- Includes helper scripts for layout checks, export, opening, and optional post-processing.

## Optional requirements

Exporting requires draw.io Desktop to be installed. The skill still works without it by producing `.drawio` files you can open manually.

CLI locations checked by the helper:

- `drawio` or `draw.io` on `PATH`
- macOS: `/Applications/draw.io.app/Contents/MacOS/draw.io`
- WSL2 default and per-user Windows install paths

## Layout

```text
SKILL.md       Skill instructions
scripts/       Export/open/check helpers
references/    XML and layout references
```
