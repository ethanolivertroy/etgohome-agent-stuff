# Workflow recipes

Read when applying a packaged prompt shortcut or orchestrating planning, implementation, review, research, context gathering, clarification, handoffs, or cleanup.

## Applying Prompt Techniques Without Slash Commands

The packaged prompt templates encode workflows the parent agent can run on demand. If the user provides a URL, issue, PR, plan, local file, screenshot, or freeform target, treat that target as the primary scope: read or fetch it before launching children, then include it explicitly in every child task. Do not depend on the parent conversation history when the recipe calls for fresh context.

### Parallel review technique

Use this when the user wants adversarial review of a diff, plan, issue, file, or implemented work. Launch fresh-context `reviewer` agents with distinct angles generated from the actual target. Common angles are correctness/regressions, tests/validation, and simplicity/maintainability; adapt for TypeScript, UI, security, docs, or large structural changes. Reviewers should inspect files and diffs directly, return concise evidence-backed findings with file/line references, and avoid edits unless the user explicitly asks for a writer pass. The parent synthesizes fixes worth doing now, optional improvements, and feedback to ignore/defer before applying anything.

### Review-loop technique

Use this when the user wants implementation or current diff review to continue until reviewers stop finding fixes worth doing now. Keep the loop in the parent session: one async `worker` implements or fixes, fresh-context `reviewer` agents inspect the actual repo and diff, the parent synthesizes accepted fixes, and one async forked `worker` applies them. The parent can express the sequence up front as an async/background chain when the workflow is known, or continue with explicit follow-up subagent runs after each async completion. For an initial chain, pass `async: true` so the main chat is unblocked; do not set `clarify: true` unless the user explicitly wants the foreground clarify UI. Treat an async implementation worker handoff as an intermediate state, not final completion, unless the user explicitly asked for worker-only work, review-only output, or to stop after implementation. Stop when reviewers find no blockers or fixes worth doing now, remaining feedback is optional or deferred, an unapproved product/scope/architecture decision appears, or the max review-round cap is reached. Default to 3 review rounds unless the user sets a different cap. Do not loop for optional polish, and do not let children launch subagents or decide the loop outcome.

### Parallel research technique

Use this when the question needs both external evidence and local implications. Combine `researcher` for official docs, specs, ecosystem behavior, recent changes, benchmarks, and primary sources with `scout` for repository files, patterns, constraints, tests, and likely integration points. Give each child a distinct angle: external evidence, local code context, and practical tradeoffs. Ask for source links or file ranges, confidence level, gaps, and decision implications. Do not ask these children to edit unless implementation was explicitly requested.

### Parallel context-build technique

Use this before planning or implementation when a stronger handoff is needed. Run a chain with one parallel step of `context-builder` agents rather than top-level parallel tasks, so relative output files live under the temporary chain directory. Give every task a distinct output path such as `context-build/request-and-scope.md`, `context-build/codebase-and-patterns.md`, and `context-build/validation-and-risks.md`. Choose two or three builders: request/scope, codebase/patterns, and validation/risks. Each builder must read every relevant file needed to understand its slice, follow imports/callers/tests/docs/config, conduct tool-available web research when needed, and include a compact `meta-prompt` section. The parent synthesizes the outputs into important context, recommended next meta-prompt, open questions, assumptions, and artifact paths.

Example shape:

```typescript
subagent({
  chain: [{
    parallel: [
      { agent: "context-builder", task: "Build request/scope context for: ...", output: "context-build/request-and-scope.md" },
      { agent: "context-builder", task: "Build codebase/pattern context for: ...", output: "context-build/codebase-and-patterns.md" },
      { agent: "context-builder", task: "Build validation/risk context for: ...", output: "context-build/validation-and-risks.md" }
    ]
  }],
  context: "fresh"
})
```

### Parallel handoff-plan technique

Use this when the user needs a solution brief or implementation-ready handoff from an external reference plus local code context, such as “study this library behavior, inspect our codebase, then produce a worker prompt.” Run a chain with a first parallel group and a second synthesis `context-builder` step. The first group usually includes `researcher` for external projects/docs/prompt guidance and `context-builder` for local code context; add a second `context-builder` for implementation strategy only when the scope is large enough to benefit. Use distinct output paths under `handoff/`, then have the synthesis `context-builder` read those outputs and write `handoff/final-handoff-plan.md` with the recommended approach, likely files, constraints, non-goals, validation, risks, unresolved questions, and final compact implementation-ready meta-prompt.

Example shape:

```typescript
subagent({
  chain: [
    { parallel: [
      { agent: "researcher", task: "Research the external reference and transferable implementation ideas for: ...", output: "handoff/external-reference.md" },
      { agent: "context-builder", task: "Build local codebase context for: ...", output: "handoff/local-context.md" },
      { agent: "context-builder", task: "Compare evidence and propose implementation strategy for: ...", output: "handoff/implementation-strategy.md" }
    ] },
    { agent: "context-builder", task: "Read {previous} and synthesize the final handoff plan and implementation-ready meta-prompt.", output: "handoff/final-handoff-plan.md" }
  ],
  context: "fresh"
})
```

### Gather-context-and-clarify technique

Use this at the start of non-trivial work. Launch `scout` for local context and `researcher` only when external docs, recent sources, ecosystem context, or primary evidence would materially improve understanding. Ask children for concise findings plus remaining clarification questions. Then synthesize what is known and use `interview` to ask the unresolved questions needed for shared understanding before planning or implementing.

### Parallel cleanup technique

Use this after implementation when the user wants cleanup review or when a final pass would reduce AI-slop. Launch two fresh-context `reviewer` tasks with `output: false` and `progress: false`: one deslop pass and one verbosity pass. If the `deslop` or `verbosity-cleaner` skills are available, pass the relevant skill to that reviewer; otherwise inline the criteria. Both reviewers are review-only and should flag concrete issues with severity, file/line references, and smallest safe fixes. Review-only/no-edit beats progress-writing or artifact-writing instructions. The parent decides what to apply and asks before making changes unless cleanup was already authorized.

## Prompt Template Integration

The package includes prompt shortcuts for common workflows: `/parallel-review`,
`/review-loop`, `/parallel-research`, `/parallel-context-build`,
`/parallel-handoff-plan`, `/gather-context-and-clarify`, and
`/parallel-cleanup`. Use them when the user wants repeatable review,
review/fix loops, research, context handoff, implementation handoff,
clarification, or cleanup-review patterns. `/parallel-review autofix` and
`/parallel-cleanup autofix` synthesize reviewer feedback and then apply only the
fixes worth doing now. Parent agents can also apply the same recipes directly
with `subagent(...)` when the user describes the workflow in natural language
instead of invoking a slash command.

If `pi-prompt-template-model` is installed, additional user prompt templates can delegate into
`pi-subagents`. This is useful when a slash command should always run through a
particular agent or with forked context.

## Common Workflows

### Recon → Plan → Implement

```typescript
subagent({
  chain: [
    { agent: "scout", task: "Map the auth flow and summarize relevant files" },
    { agent: "planner", task: "Plan the migration from {previous}" },
    { agent: "worker", task: "Implement the approved plan from {previous}" }
  ]
})
```

### Clarify → Plan → Implement → Review (self-orchestrated workflow)

When you are the orchestrating agent for a new feature or non-trivial change, factor in the packaged prompt workflows without literally invoking slash commands. Use the same patterns through tools and subagents.

Keep builtin agent defaults unless the user explicitly asks for a different model, thinking level, skills, output behavior, context mode, or other override. Do not add overrides just because you are orchestrating; the defaults encode the intended role behavior. In particular, packaged `planner`, `worker`, and `oracle` default to forked context.

When the user approves launching a subagent to carry out a plan or workflow, treat that as approval to generate a proper role-specific meta prompt for that subagent. Include the approved plan path or summary, clarified requirements, non-goals, relevant context, role boundaries, files or areas to inspect, acceptance criteria, expected output, and validation expectations. Do not pass vague instructions like “implement the plan fully” or “review this” by themselves.

- `/gather-context-and-clarify` maps to: launch `scout` and, when needed, `researcher`; synthesize findings; then use `interview` to ask every clarification question needed for shared understanding.
- `/parallel-review` maps to: launch fresh-context `reviewer` agents with distinct review angles; synthesize the feedback before applying anything.
- `/review-loop` maps to: keep the parent in charge of worker → fresh reviewers → synthesized fix worker cycles until no fixes worth doing now remain, an unapproved decision appears, or the review-round cap is reached.
- `/parallel-research` maps to: combine local `scout` context with external `researcher` evidence when current docs, ecosystem behavior, or API details matter.
- `/parallel-context-build` maps to: run a chain-mode parallel group of `context-builder` agents with distinct temp output paths, then synthesize their context and meta-prompt sections.
- `/parallel-handoff-plan` maps to: run external `researcher` plus local/strategy `context-builder` passes, then a synthesis `context-builder` that writes an implementation handoff plan and implementation-ready meta-prompt.
- `/parallel-cleanup` maps to: use review-only cleanup passes after implementation, especially for simplicity, verbosity, and redundant tests.

For feature work, use this sequence as scaffolding for parent-agent behavior:

```text
clarify → validation contract → planner → async worker → parallel async fresh-context reviewers/validators → async fix worker → follow-up review when warranted → parent review
```

The validation contract defines what done means before code is written: expected behavior, acceptance checks, commands or user flows to exercise, and evidence the worker should return. Keep it lightweight for small tasks, but make it explicit enough that reviewers and validators are checking the intended outcome rather than the worker’s own assumptions.

The first `worker` implements the approved plan. The parent continues with independent inspection or validation prep while it runs, not parallel edits to the same worktree. When the async worker completes, treat its handoff as the transition into review, not as final completion, unless the user explicitly asked for worker-only work, review-only output, or to stop after implementation. Parallel reviewers inspect the resulting diff from fresh context. Validators check behavior with the best available evidence: commands, tests, browser/CLI interaction, screenshots, logs, or manual reproduction notes. The final `worker` applies synthesized review fixes in forked context, then the parent looks over the final diff before completing. The parent may launch these steps as an initial async chain when the workflow is already clear, or as follow-up subagent runs after each async completion. Initial chains should pass `async: true` so the main chat is unblocked; avoid `clarify: true` unless the user asked for foreground clarification. Do not stop after parallel review unless the user explicitly asked for review-only output or the review surfaced a decision that needs approval first.

For complex work, risky changes, broad refactors, or many changed lines, increase review and validation fanout rather than trusting one reviewer. Use distinct angles such as correctness/regressions, tests/validation, simplicity/maintainability, security/privacy, performance, docs/API contracts, and user-flow behavior. When reviewers find non-trivial issues or the fix worker touches many lines, run another focused review round before final validation.

For very large work, split into serial milestones instead of launching a swarm of writers. Each milestone gets one writer, a validation contract, fresh-context review/validation, a fix pass, and parent acceptance before the next milestone starts. Use parallel subagents inside a milestone for read-only context, research, review, and validation only.

Keep orchestration authority in the parent session. Child subagents should not launch more subagents, read this skill, or run their own orchestration loops. Spawned subagents do not receive the `pi-subagents` skill, parent-only status/control/slash messages, prior parent `subagent` tool-call/tool-result artifacts, or the `subagent` extension tool. Child context filtering also strips old hidden orchestration-instruction messages when they appear in inherited history. Every child also receives a boundary instruction that says the parent owns orchestration, the child must not propose or run subagents, and implementation children must call real edit/write tools instead of printing pseudo tool calls. Pass children concrete role-specific work instead.

1. Clarify first. This is mandatory. Gather code context with `scout` or `context-builder`, add `researcher` only when external evidence matters, then ask the user clarifying questions with `interview` until scope, acceptance criteria, constraints, and non-goals are clear.
2. Define the validation contract. State what done means before implementation: expected behavior, checks to run, user flows to exercise, and evidence required in the worker handoff. For UI, CLI, integration, or workflow changes, include at least one validator angle that uses the product the way a user would rather than only reading code.
3. Plan when useful. For complex work, call `planner` or write a plan doc yourself and get approval before implementation. For simple work, confirm shared understanding and explicitly note why planning is skipped.
4. Implement with one writer. After approval, launch `worker` asynchronously with a proper meta prompt that includes clarified requirements, relevant context, plan path or summary, the validation contract, and output expectations. Packaged `worker` defaults to forked context; pass `context: "fresh"` only when you intentionally want a fresh child. While it runs, prepare validation or inspect adjacent code instead of editing the same worktree.
5. Require a useful worker handoff. Ask the worker to report changed files, what was implemented, what was left undone, commands run with exit codes, validation evidence, surprises or new risks, decisions made inside approved scope, and decisions needing parent approval.
6. Review after implementation. After the worker completes, launch parallel async fresh-context `reviewer` agents for correctness/regressions, tests/validation, and simplicity/maintainability. Add security, performance, docs/API, domain-specific, or user-flow validators for complex work, risky changes, broad refactors, or many changed lines. Use `output: false` unless review artifacts are explicitly needed.
7. Synthesize, then run the fix worker. Separate blockers, fixes worth doing now, optional improvements, and feedback to ignore/defer, then launch an async forked `worker` to apply fixes worth doing now when the workflow is implementation-authorized. If reviewers found scope/product/architecture choices that were not approved, ask the user first instead of applying them.
8. Review again when warranted. If the fix worker made substantial changes or addressed non-trivial findings, run another focused parallel review round before final validation.
9. Validate and complete. After the fix worker and any follow-up review return, inspect the final diff yourself, run or confirm focused validation, update docs/changelog when relevant, and summarize what changed and why.

Example implementation handoff after clarification and optional planning:

```typescript
subagent({
  agent: "worker",
  task: "Implement the approved feature.\n\nClarified requirements:\n- ...\n\nPlan: see ~/Documents/docs/...-plan.md\n\nValidation contract:\n- ...\n\nReturn a handoff with changed files, what was implemented, what was left undone, commands run with exit codes, validation evidence, surprises/new risks, and decisions needing parent approval.",
  async: true
})
```

Example review pass after implementation:

```typescript
subagent({
  tasks: [
    { agent: "reviewer", task: "Review the current diff for correctness and regressions. Inspect changed files directly; do not rely on the worker's reasoning.", output: false },
    { agent: "reviewer", task: "Review the current diff for tests and validation quality against the validation contract. Inspect changed files directly.", output: false },
    { agent: "reviewer", task: "Review the current diff for simplicity and maintainability. Inspect changed files directly.", output: false }
  ],
  concurrency: 3,
  context: "fresh",
  async: true
})
```

Example fix worker after parallel reviews:

```typescript
subagent({
  agent: "worker",
  task: "Apply the synthesized reviewer feedback below. Only apply fixes worth doing now; preserve user-approved scope; ask before unapproved product or architecture changes. Run focused validation and summarize what changed.\n\nReviewer synthesis:\n...",
  async: true
})
```

### Review loop

Do not treat review as the final step for implementation work. Run reviewers and validators, synthesize their findings against user scope and the validation contract, then launch one `worker` for accepted fixes when implementation is authorized.

When an async implementation worker completes, treat the worker handoff as an intermediate state. The next parent action is review fanout, then synthesis, then a fix worker if reviewers found fixes worth doing now. This can be planned as an initial async chain when the whole workflow is known, or continued as follow-up subagent runs when the parent only launched the first worker initially. Initial chains should pass `async: true` so the main chat is unblocked; `clarify: true` is the explicit foreground opt-in.

For explicit review-loop requests, repeat worker → fresh-reviewer → synthesized-fix-worker cycles until reviewers find no blockers or fixes worth doing now, remaining feedback is optional or intentionally deferred, an unapproved product/scope/architecture decision needs the user, or the max review-round cap is reached. Default to 3 review rounds unless the user sets a different cap. For complex work, many changed lines, or any fix pass that materially changes the diff, run another focused review round before the parent’s final look; otherwise stop instead of chasing optional polish.

### Parallel non-conflicting analysis

```typescript
subagent({
  tasks: [
    { agent: "scout", task: "Audit frontend auth flow" },
    { agent: "researcher", task: "Research current retry/backoff best practices" }
  ]
})
```

### Saved chain

```text
/run-chain review-chain -- review this branch
```

Use saved `.chain.md` workflows when the user wants a repeatable multi-agent flow without rewriting the chain each time.
