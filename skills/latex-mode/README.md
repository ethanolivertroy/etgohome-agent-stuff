# LaTeX Mode

A manually invoked skill that turns technical explanations into explicit, checkable LaTeX
derivations: definitions, assumptions, derivation steps, a counterexample attempt, and a
conclusion.

## Inspiration

Inspired by [Can Bölük's public post](https://x.com/_can1357/status/2094619896063635630)
suggesting that asking a model to reply in LaTeX can replace conversational model prose with a
denser formal presentation.

This is an original skill built from that idea. It does not copy the post's technical content or
claim that formatting exposes a model's hidden chain-of-thought.

## Usage

The skill only runs when invoked explicitly (`disable-model-invocation: true`):

```text
/latex-mode Formalize the safety and liveness properties of this queue implementation.
```

It defaults to a standalone LaTeX document, or rendered Markdown with LaTeX equations on request.
Compile the output with any standard LaTeX engine to check it.

Install it with the rest of [etgohome-agent-stuff](../../README.md#install).
