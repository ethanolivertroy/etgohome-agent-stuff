---
name: latex-mode
description: Use only when the user explicitly requests LaTeX mode or a technical answer formalized as a checkable LaTeX derivation.
metadata:
  version: "0.1.0"
  inspiration: "https://x.com/_can1357/status/2094619896063635630"
---

# LaTeX Mode

Turn a technical answer into a formal artifact whose assumptions, definitions, derivation, and verification can be inspected directly.

This does not reveal hidden chain-of-thought. Produce a concise, user-facing derivation with enough intermediate structure to check the result. Never claim access to private reasoning, internal scratch work, token probabilities, or unrevealed model state.

## When to use

Invoke `/latex-mode` when the user wants:

- A mathematical derivation, proof sketch, algorithm analysis, or complexity argument.
- A protocol, state machine, concurrency model, allocation rule, or data invariant formalized.
- A security property expressed through assumptions, adversary capabilities, preconditions, and proof obligations.
- An architecture or design decision stripped of conversational filler and reduced to explicit constraints and consequences.
- A dense technical explanation that distinguishes facts, assumptions, hypotheses, and unresolved questions.

Do not use it for routine status updates, casual writing, or simple edits that do not benefit from formalization.

## Execution rule

Formatting is not a substitute for work. If the request requires repository inspection, calculation, testing, debugging, or file changes, perform and verify that work first. Use LaTeX to report the grounded result.

## Method

1. **Scope the claim.** State the exact question, desired result, and boundary of the analysis. If essential information is unavailable, mark it as unknown instead of inventing it.
2. **Build the vocabulary.** Define every symbol, set, function, state, event, and predicate before using it. Keep notation minimal.
3. **Classify premises.** Separate verified facts, user-provided assumptions, working hypotheses, and external constraints. Cite code paths, tests, measurements, or sources when available.
4. **Choose the smallest useful formalism.**
   - Algorithms: inputs, outputs, recurrence or transition, invariant, complexity.
   - Stateful systems: state space, events, guards, transition function, safety and liveness properties.
   - Security: assets, trust boundaries, adversary capabilities, preconditions, security properties, counterexamples.
   - Design decisions: options, hard constraints, observed evidence, consequences. Do not fabricate numerical scores or weights.
5. **Derive the result.** Present numbered claims or equations. Each transformation must follow from a named premise, definition, observation, or previously established claim.
6. **Try to break it.** Check boundary conditions, counterexamples, units, degenerate inputs, races, failure states, and contradictions with known evidence.
7. **Conclude narrowly.** State what follows, what does not follow, and which uncertainties remain.

## Output contract

Default to a standalone, compilable LaTeX document. Return the LaTeX source directly without wrapping it in a Markdown code fence. Use this preamble unless the task needs something else:

```latex
\documentclass[11pt]{article}
\usepackage[T1]{fontenc}
\usepackage{amsmath,amssymb,amsthm}
\usepackage{booktabs}
\newtheorem{definition}{Definition}
\newtheorem{proposition}{Proposition}
\newtheorem{invariant}{Invariant}
\newtheorem{assumption}{Assumption}
\begin{document}

\section{Question}
% Exact claim and scope.

\section{Definitions and premises}
% Symbols, verified facts, assumptions, and hypotheses.

\section{Derivation}
% Numbered steps with explicit justification.

\section{Verification}
% Edge cases, counterexamples, and evidence checks.

\section{Conclusion}
% Narrow result and unresolved uncertainty.

\end{document}
```

If the user asks for an answer rendered in chat rather than compilable source, use short Markdown headings with LaTeX math delimiters instead. If code is essential, include only the smallest relevant excerpts and name their source paths.

## Style constraints

- Prefer definitions, propositions, cases, aligned equations, transition rules, tables, and pseudocode over rhetorical narration.
- Use prose where prose is clearer. Never invent decorative equations to make an argument appear rigorous.
- Avoid slogans, fake quotations, generic praise, and model-style filler.
- Preserve uncertainty with explicit labels such as `Assumption`, `Hypothesis`, `Not established`, and `Counterexample`.
- Keep the derivation proportional to the question. Formal does not mean long.
- Never expose secrets or reproduce sensitive source material merely to make the document self-contained.

## Verification checklist

Before returning the answer, verify that:

- Every symbol is defined before use.
- Every conclusion traces to stated premises or evidence.
- Facts, assumptions, and hypotheses are distinguishable.
- At least one failure mode, edge case, or counterexample was considered.
- The conclusion does not overclaim hidden reasoning or certainty.
- The LaTeX delimiters and environments are balanced.
- The response still completes the user's actual task rather than only reformatting it.
