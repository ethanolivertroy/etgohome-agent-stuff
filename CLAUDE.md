# etgohome-agent-stuff

Public repo of agent skills, Pi extensions, and prompts. It is both a Pi package
(`package.json` → `pi` key) and a Claude Code marketplace whose single plugin, `etgohome`,
has the repo root as its plugin root.

- `skills/` holds portable Agent Skills only. Claude Code auto-loads everything in it.
- Anything that needs a Pi extension or Pi-only tools goes in `pi/skills/` or `pi/prompts/`.
- Never add root-level `commands/`, `agents/`, or `hooks/` folders unless they are meant for
  Claude Code; it auto-loads them too.
- Bump `version` in `.claude-plugin/plugin.json` when portable skills change.
- Before committing: `claude plugin validate .claude-plugin/marketplace.json`,
  `claude plugin validate .claude-plugin/plugin.json`, and `npm run secrets:scan`.
- Everything here is public: no secrets, tokens, or private hostnames.
