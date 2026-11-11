# etgohome-agent-stuff

Public repo of portable Agent Skills, Pi extensions, and prompts. `AGENTS.md` holds
shared repository instructions. Each skill uses the Agent Skills `SKILL.md` format.

- `skills/` holds portable Agent Skills with their scripts, references, and assets.
- Anything that needs a Pi extension or Pi-only tools goes in `pi/skills/` or `pi/prompts/`.
- `extensions/` holds Pi extensions; `package.json` lists Pi package content under `pi`.
- Keep portable skills independent of a specific agent's plugin or marketplace packaging.
- Follow the [Agent Skills specification](https://agentskills.io/specification) for
  `SKILL.md` frontmatter; put extra descriptive fields under string-valued `metadata`.
- Before committing, run `npm run secrets:scan`.
- Everything here is public: no secrets, tokens, or private hostnames.
