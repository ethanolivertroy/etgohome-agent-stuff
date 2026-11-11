# etgohome-agent-stuff

My public Agent Skills, Pi extensions, and prompts for coding agents.

- Inspired by https://github.com/mitsuhiko/agent-stuff
- Consolidates `my-agent-stuff`, `pi-drawio`, `emass-skills`, `mischief-managed`, and `latex-mode` from `etgohome-plugin-staging`
- I don't marry any one tool, but [pi](https://pi.dev) is mostly my daily driver now

## Install

**Pi** (everything: extensions, skills, prompts):

```bash
pi install git:github.com/ethanolivertroy/etgohome-agent-stuff
```

**Agent Skills-compatible tools** (Codex, Claude Code, etc.): copy or symlink selected
folders from `skills/` into your tool's skills directory. Each folder includes a
`SKILL.md` and any supporting scripts, references, or assets.

The portable skills follow the [Agent Skills format](https://agentskills.io/specification).
Shared instructions for agents working on this repository live in [`AGENTS.md`](./AGENTS.md).

## Layout

```
AGENTS.md                     # shared repository instructions for coding agents
skills/                       # portable Agent Skills, each with a SKILL.md
pi/skills/                    # Pi-only skills (depend on extensions in this repo)
pi/prompts/                   # Pi prompt templates (use the subagents extension)
extensions/                   # Pi extensions
package.json                  # Pi package manifest (`pi` key)
```

Skills and prompts that depend on Pi tools belong under `pi/`. See
[upstream package sources](./docs/upstream-pi-packages.md)
for provenance and license notes.

## Secret scanning

This public repo uses Gitleaks via repo-managed Git hooks:

```bash
brew install gitleaks
npm run secrets:install-hooks
npm run secrets:scan
```

Installed hooks run a staged scan before commits and an outgoing-commit scan before pushes. Use `SKIP_SECRET_SCAN=1` only for emergency bypasses.

Extensions that need API tokens should use the shared local resolver in [`src/secret-resolver.ts`](./src/secret-resolver.ts); see [`docs/extension-secret-management.md`](./docs/extension-secret-management.md). Store password-manager refs in `.env.local`, not raw tokens.

## Pi Extensions

| Extension | Description |
| --- | --- |
| [exa-search](./extensions/exa-search) | Advanced Exa web search with categories, filters, and optional result contents. |
| [linear-mcp](./extensions/linear-mcp) | Mirror Linear's official hosted MCP server into Pi tools with dynamic `tools/list` registration. |
| [ollama-web-search](./extensions/ollama-web-search.ts) | Local Ollama-backed `web_search` and `web_fetch` tools. |
| [grok-cli](./extensions/grok-cli) | Use Grok Build/Composer in Pi through the existing Grok CLI login session. |
| [hf-buckets](./extensions/hf-buckets) | Safe private Hugging Face Bucket backups with detached jobs, verification, and quarantine. |
| [mm-command](./extensions/mm-command.ts) | Shortcut command for invoking the `mischief-managed` session-note skill. |
| [nanobanana](./extensions/nanobanana) | Generate or edit images with Google Gemini image models. |
| [rich-clipboard](./extensions/rich-clipboard) | Copy Markdown, HTML, or plain text to the system clipboard for rich paste targets. |
| [termdraw](./extensions/termdraw) | Open termDRAW inside a Pi overlay with `/termdraw`. |
| [voipi](./extensions/voipi) | Text-to-speech tools and `/tts` commands. |
| [subagents](./extensions/subagents) | Subagent orchestration tool, built-in agents, and related commands. |
| [autoresearch](./extensions/autoresearch) | Experiment loop tools, dashboard, hooks, and autoresearch command. |

## Skills

Portable skills for Agent Skills-compatible runtimes:

| Skill | Description |
| --- | --- |
| [drawio](./skills/drawio) | Generate native draw.io diagrams and export them to PNG, SVG, PDF, or JPG. |
| [emasser-*](./skills/emasser-setup) | Six skills for the MITRE eMASSer CLI: setup, get, post, put, delete, and artifacts. See [docs/emasser.md](./docs/emasser.md). |
| [elements-of-style](./skills/elements-of-style) | Edit or write plain, concise English with Strunk's *Elements of Style* rules (outdated 1920 rules flagged). |
| [exif-stripper](./skills/exif-stripper) | Strip sensitive EXIF metadata from images before publishing to the web. |
| [iac-security-scanner](./skills/iac-security-scanner) | Scan Terraform, Kubernetes, CloudFormation, ARM templates, and Dockerfiles for security misconfigurations with NIST mappings. |
| [image-generator](./skills/image-generator) | Generate and edit images through Gemini image models. |
| [jot](./skills/jot) | Append quick notes to a configured Obsidian daily note. |
| [latex-mode](./skills/latex-mode) | Turn a technical answer into a checkable LaTeX derivation (invoke with `/latex-mode`). |
| [linear](./skills/linear) | Manage Linear issues, projects, cycles, and status workflows through the Linear MCP server. |
| [made-to-stick](./skills/made-to-stick) | Apply the SUCCESs framework to make ideas, copy, and content more memorable. |
| [mesh-security](./skills/mesh-security) | Analyze Istio, Consul, and Linkerd service mesh configurations for security issues and NIST control mappings. |
| [mischief-managed](./skills/mischief-managed) | Save a lean end-of-session note (Obsidian, Notion, or any Markdown folder) and keep a Marauder's Map index. |
| [security-review](./skills/security-review) | Perform high-signal security reviews of pending branch changes with explicit false-positive filtering. |

Pi-only skills:

| Skill | Description |
| --- | --- |
| [clipboard-posting](./pi/skills/clipboard-posting) | Use the rich clipboard extension for approved drafts, LinkedIn, and X posting flows. |
| [conversation-dump](./pi/skills/conversation-dump) | Export a full readable Markdown transcript of the current Pi conversation. |
| [huggingface-backup](./pi/skills/huggingface-backup) | Safely back up local folders to private Hugging Face Buckets using `hf` and the `hf-buckets` extension. |
| [pi-subagents](./pi/skills/pi-subagents) | Instructions for delegating work to the local subagent extension. |
| [autoresearch-create](./pi/skills/autoresearch-create) | Start an autonomous optimization experiment loop. |
| [autoresearch-finalize](./pi/skills/autoresearch-finalize) | Finalize autoresearch sessions into reviewable branches. |
| [autoresearch-hooks](./pi/skills/autoresearch-hooks) | Author pre/post iteration hooks for autoresearch. |

## Pi Prompts

Prompt templates in [`pi/prompts/`](./pi/prompts) for the subagents extension: `parallel-review`,
`parallel-cleanup`, `parallel-research`, `parallel-context-build`, `parallel-handoff-plan`,
`gather-context-and-clarify`, and `review-loop`.

## License

[MIT](./LICENSE), except where an upstream license file says otherwise.
