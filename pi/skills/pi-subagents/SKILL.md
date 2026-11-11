---
name: pi-subagents
description: |
  Delegate work to builtin or custom subagents with single-agent, chain,
  parallel, async, forked-context, and intercom-coordinated workflows. Use
  for advisory review, implementation handoffs, and multi-step tasks where a
  single agent should stay in control while other agents contribute context,
  planning, or execution.
---

# Pi Subagents

This skill is for the main parent orchestrator only. Do not inject or follow it inside spawned child subagents. The parent session owns delegation, orchestration, review fanout, and final fix-worker launches; child subagents should receive concrete role-specific tasks and should not run their own subagent workflows.

Use this skill when the parent orchestrator needs to launch a specialized subagent, compose multiple agents into a workflow, or create/edit agents and chains on demand.

## When to Use

- **Advisory review**: use fresh-context `reviewer` agents for adversarial code review, or fork to `oracle` when inherited decisions and drift matter
- **Implementation handoff**: have `oracle` advise, then `worker` implement only after an approved direction
- **Recon and planning**: use `scout` or `context-builder`, then `planner`
- **Parallel exploration**: run multiple non-conflicting tasks concurrently
- **Long-running work**: launch async/background runs and inspect them later
- **Subagent control**: watch needs-attention signals and soft-interrupt only when a delegated run is genuinely blocked
- **Agent authoring**: create, update, or override agents and chains for a project

## Tool vs Slash Commands

Agents can use the `subagent(...)` tool directly for execution, management, status, and control.
Humans often use the slash-command layer instead:

- `/run` — launch a single agent
- `/chain` — launch a chain of steps
- `/parallel` — launch top-level parallel tasks
- `/run-chain` — launch a saved `.chain.md` workflow
- `/subagents-doctor` — diagnose setup, discovery, async paths, and intercom bridge state

Prefer the tool when you are writing agent logic. Prefer the slash commands when
you are guiding a human through an interactive flow.

Packaged prompt shortcuts are also available for repeatable workflows. Treat them as reusable orchestration recipes, not just human slash commands. When the user asks for one of these shapes, or when the workflow clearly fits, apply the same pattern directly with `subagent(...)` and other tools:
- `/parallel-review` — fresh-context reviewers with distinct review angles, then synthesis
- `/review-loop` — parent-orchestrated worker, fresh-reviewer, and fix-worker cycles until clean or capped
- `/parallel-research` — combine `researcher` and `scout` for external evidence plus local code context
- `/parallel-context-build` — parallel `context-builder` passes that produce planning handoff context and meta-prompts
- `/parallel-handoff-plan` — external-reference research plus local `context-builder` passes, followed by a synthesis handoff plan and implementation-ready meta-prompt
- `/gather-context-and-clarify` — scout/research first, then ask the user clarifying questions with `interview`
- `/parallel-cleanup` — two fresh-context reviewers (deslop + verbosity passes) for an adversarial cleanup review of the current diff

## Read the matching guide

Read the relevant guide before carrying out one of these branches:

- [Workflow recipes](references/workflows.md): read when planning or implementing a feature, preparing a handoff, running review or cleanup passes, gathering research or context, clarifying requirements, or applying any packaged prompt shortcut. It contains the clarification, validation, review-loop, and completion rules for those workflows.
- [Execution modes and coordination](references/execution-and-coordination.md): read when composing parallel runs or chains, configuring saved-file output, using the clarify TUI or worktree isolation, seeking oracle advice, coordinating through intercom, or diagnosing launch and runtime failures.
- [Agent settings and authoring](references/agent-authoring.md): read when listing agents and chains, inspecting agent configuration, changing persistent builtin overrides, creating, updating, or deleting agents, or authoring agent files and their configuration fields.

## Builtin Agents

Builtin agents load at the lowest priority. Project agents override user agents,
and user/project agents override builtins with the same name.

| Agent | Purpose | Model | Typical output / role |
|-------|---------|-------|------------------------|
| `scout` | Fast codebase recon | inherits default | Writes `context.md` handoff material |
| `planner` | Creates implementation plans | inherits default | Writes `plan.md` |
| `worker` | Implementation and approved oracle handoffs | inherits default | Single-writer implementation with decision escalation |
| `reviewer` | Review-and-fix specialist | inherits default | Can edit/fix reviewed code |
| `context-builder` | Requirements/codebase handoff builder | inherits default | Writes structured context files |
| `researcher` | Web research brief generator | inherits default | Writes `research.md` |
| `delegate` | Lightweight generic delegate | inherits default | No fixed output; generic delegated work |
| `oracle` | Decision-consistency advisory review | inherits default | Advisory review, intercom coordination |

Builtin agents inherit the current Pi default model unless a run, user setting, or project setting overrides `model`. Override builtin defaults before copying full agent files when a small tweak is enough.

For one run, use inline config:

```text
/run reviewer[model=anthropic/claude-sonnet-4] "Review this diff"
```

For persistent tweaks, edit `subagents.agentOverrides` in user or project settings. User overrides apply everywhere. Project overrides apply only in that repo and win over user overrides.

## Prompting role subagents

Builtin role agents inherit the current Pi default model unless you override them. When launching them, write the task prompt as a compact contract, not a long procedural script. Define the destination and let the role choose the efficient path.

A strong subagent prompt usually includes:
- **Goal**: the concrete outcome the child should produce.
- **Context/evidence**: relevant plan paths, files, diffs, decisions, or user constraints already approved.
- **Success criteria**: what must be true before the child can finish.
- **Hard constraints**: true invariants only, such as no edits for review-only tasks, one writer thread, child must not run subagents, or escalation for unapproved decisions.
- **Validation**: targeted checks to run, or the next-best check when validation is impossible.
- **Output**: the expected summary shape, artifact path, or finding format.
- **Stop rules**: when to ask via `intercom`, when to stop after enough evidence, and when not to keep searching.

Avoid carrying over old prompt habits that over-specify every step. Use `must`, `always`, and `never` for real invariants; for judgment calls, give decision rules. For example, tell a reviewer to inspect the staged diff directly and report only evidence-backed findings, rather than prescribing every file or command. Tell a researcher the retrieval budget: start with broad targeted searches, fetch only the strongest sources, search again only when a required fact is missing, then stop.

For implementation handoffs, name the approved scope and success criteria more clearly than the process. Good prompts say what to change, what not to change, where the evidence lives, how to validate, and when to escalate. They should not ask the child to create another subagent plan or continue the parent conversation.

## Discovery and Scope Rules

Agent files can live in:
- `~/.pi/agent/agents/**/*.md` — user scope
- `.pi/agents/**/*.md` — canonical project scope
- legacy `.agents/**/*.md` — still read for compatibility, but `.pi/agents/` wins on conflicts

Chains live in:
- `~/.pi/agent/chains/**/*.chain.md` — user scope
- `.pi/chains/**/*.chain.md` — project scope

Discovery is recursive. `.chain.md` files do not define agents. Agents and chains can set optional frontmatter `package: code-analysis`; `name: scout` plus `package: code-analysis` registers as runtime name `code-analysis.scout` while serialization keeps `name` and `package` separate.

Precedence is by parsed runtime name:
1. project scope
2. user scope
3. builtin agents

## Running Subagents

### Single agent

```typescript
subagent({
  agent: "oracle",
  task: "Review my current direction and challenge assumptions."
})
```

### Forked context

```typescript
subagent({
  agent: "oracle",
  task: "Review my current direction and challenge assumptions."
})
```

`context: "fork"` creates a branched child session from the current persisted
parent session. It does **not** create a fresh minimal review context or filter
history down to only the relevant parts. Use it when you want a separate review
or execution thread that can still reference the parent session history.

### Async/background

Prefer async mode for every subagent launch. Set `async: true` no matter the task unless there is a specific reason to opt into a foreground/blocking run. This applies to scouts, researchers, workers, reviewers, validators, oracle checks, one-off delegates, chains, and parallel groups. Keep the write path single-threaded even when the run is async.

Async does not mean parallel writes. Do not edit the same active worktree while an async worker is changing it. Parent-side overlap should be reading, validation prep, synthesis, command planning, or review of unaffected context unless the writer is isolated in a separate worktree.

Do not end your turn immediately after launching an async child if you promised to keep working. Continue the local inspection, synthesis, or validation prep, then check the async run when its result is needed. If there is no independent work left and you would only be running `sleep` or status polling commands to wait, end your turn instead. Pi will deliver the async completion when it arrives.

```typescript
subagent({
  agent: "worker",
  task: "Run the full test suite",
  async: true
})
```

File-only output mode also works for async single runs, top-level parallel task items, sequential chain steps, and chain parallel task items. In chains, `{previous}` receives the compact saved-file reference when the prior step used file-only mode.

For review fanout where the parent continues a local audit:

```typescript
const run = subagent({
  agent: "reviewer",
  task: "Review the current diff for correctness issues. Do not edit files.",
  async: true,
  context: "fresh"
})
// Continue local inspection, then later call status with the returned id.
```

Inspect async runs with `subagent({ action: "status", id: "..." })` or `subagent({ action: "status" })` for active runs.

Use `resume` for follow-up work after a delegated run:

```typescript
subagent({ action: "resume", id: "run-id", message: "Follow up on this point." })
subagent({ action: "resume", id: "run-id", index: 1, message: "Continue reviewer 2." })
```

Resume behavior:
- If an async child is still running and reachable, `resume` sends the follow-up to that live child over intercom.
- If an async child has completed, `resume` revives it by starting a new async child from the persisted child session file.
- Multi-child async runs require `index` unless only one running child is selectable.
- Completed foreground single, parallel, and chain runs can also be revived by `index` while their run metadata remains in extension state.
- Revive starts a new child process from the old session context; it does not restart the same OS process.
- If the chosen child has no persisted `.jsonl` session file, resume fails and reports that directly.

Use diagnostics when setup or child startup looks wrong:

```typescript
subagent({ action: "doctor" })
```

Humans can use `/subagents-doctor` for the same read-only report. It checks runtime paths, discovery counts, async support, current session context, and intercom bridge state.

### Subagent control

Subagent control is the runtime visibility and intervention layer for delegated runs. It is separate from lifecycle status. Lifecycle status says whether a child is `queued`, `running`, `paused`, `complete`, or `failed`. Activity reporting is factual: it tracks the last observed activity time and the current tool when known. It does not pretend to know that a child is truly stuck.

Default behavior is intentionally conservative. When no activity has been observed past the configured threshold, the run emits a `needs_attention` control event. Foreground runs can push this as a `subagent:control-event` event, and async runs persist it to `events.jsonl` so the parent tracker can surface it without constant manual polling. Notification-worthy control events are also inserted into the visible transcript so both the user and the parent agent can see them, with a proactive hint plus concrete `nudge`, `status`, and `interrupt` options. Visible notifications fire once per child run and attention state.

Use soft interrupt when a child is clearly blocked or drifting and the parent needs to regain control:

```typescript
subagent({ action: "interrupt" })
```

Pass `id` when targeting a specific controllable run:

```typescript
subagent({ action: "interrupt", id: "abc123" })
```

A soft interrupt cancels the current child turn and leaves the run paused. It does not mean the delegated task succeeded or failed. After an interrupt, decide the next explicit action: resume with clearer instructions, replace the task, ask the user, or stop the workflow.

Per-run control thresholds can be overridden when a task legitimately runs without observable output for longer than usual:

```typescript
subagent({
  agent: "worker",
  task: "Run the slow migration test suite",
  control: {
    needsAttentionAfterMs: 300000,
    notifyOn: ["needs_attention"]
  }
})
```

If the run already has an active intercom bridge target, needs-attention notifications can also prepare a compact intercom ping for the orchestrator. When a child route is available, the ping tells the orchestrator which agent needs attention and includes the exact `intercom({ action: "send", to: "..." })` target for a nudge. Do not invent a target or ask the child to self-report when no bridge exists.

## Important Constraints

- **Forking requires a persisted parent session.** If the current session does not
  have a persisted session file, forked runs fail. Packaged `planner`, `worker`,
  and `oracle` default to forked context, so use `context: "fresh"` explicitly
  when that is not available or not wanted.
- **Forked runs inherit parent history.** They are branched threads, not fresh
  filtered contexts. Use fresh context for adversarial reviewers unless the user explicitly asks for forked context.
- **Default subagent nesting depth is 2.** Deeper recursive delegation is blocked
  unless configured otherwise.
- **Attention signals are not lifecycle state.** `needs_attention` means no activity has been observed past the configured threshold. `paused` means the child turn was intentionally interrupted or is awaiting direction; it is not the same as `failed`.
- **Intercom asks are blocking.** A session can only maintain one pending outbound
  ask wait state at a time.
- **Keep conversational authority clear.** Advisory subagents should not silently
  become second decision-makers.

## Best Practices

### Prefer async orchestration

Launch every subagent asynchronously by default. Use `async: true` for scouts, researchers, workers, reviewers, validators, oracle checks, one-off delegates, chains, and parallel groups unless you intentionally need a foreground/blocking run. The parent should keep moving: inspect code while scouts run, prepare validation while a worker implements, do a local diff pass while reviewers review, and synthesize or verify while a fix worker applies accepted feedback. Async is the default orchestration posture; foreground runs are the explicit opt-out.

### Keep writes single-threaded by default

A strong pattern is one main decision-maker plus advisory/research/review/validation subagents around it. Use `oracle` for advice and `worker` for the actual write path. Parallelize reading, review, validation, and synthesis support, not normal writes, unless you deliberately isolate writers with worktrees. A child that writes should report what changed, what was left undone, commands run with exit codes, validation evidence, surprises, and any decisions that need parent approval.

### Use fork for branched advisory or execution threads

Forked runs are useful when the child should reason in a separate thread while
still inheriting the parent’s accumulated context. They are especially useful for
`oracle`, which audits inherited decisions and drift. For adversarial code review,
prefer fresh-context reviewers that inspect the repo and diff directly unless the
user explicitly requests forked context.

### Prefer narrow tasks

Give subagents specific tasks rather than vague mandates.
`Review auth.ts for null-check gaps` works better than `Review everything`.

### Escalate decisions upward

If a subagent encounters an unapproved product, architecture, or scope choice,
it should coordinate back via `intercom` instead of deciding alone.

### Intervene only on clear control signals

Use subagent control proactively when a delegated run emits `needs_attention`, or when a human asks you to regain control. Do not interrupt just because a child has briefly produced no output. Silence can be normal during long tool calls, test runs, or model reasoning.

### Name sessions meaningfully

Use `/name` so intercom targeting stays stable.
