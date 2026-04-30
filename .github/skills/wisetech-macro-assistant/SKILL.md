---
name: wisetech-macro-assistant
description: 'Use when answering CargoWise macro questions, including macro creation, macro debugging, macro explanation, Workflow MCR conditions, Document or DocBuilder expressions, Email HTML macro safety, field selection, pattern matching, WorkflowItems, Event.Params, collections, and CargoWise macro best practices.'
argument-hint: 'Describe the macro task and target context: Workflow MCR, Document / DocBuilder, or Email HTML.'
---

# WiseTech Macro Assistant

Use this skill for CargoWise macro work that needs the same strict context handling, safety checks, and output structure as the Macro-Helper workflow.

## When To Use

- Create a new CargoWise macro from plain English.
- Debug, lint, or fix a broken macro.
- Explain an existing macro.
- Suggest a safer or cleaner macro pattern.
- Answer questions about Workflow MCR, Document / DocBuilder, or Email HTML macro syntax.
- Answer questions about field choice, pattern matching, WorkflowItems, Event.Params, collection usage, or macro best practice.

## Procedure

1. Determine the user's intent: creation, debugging, explanation, or pattern suggestion.
2. Determine the macro context before producing syntax: Workflow MCR, Document / DocBuilder, or Email HTML.
3. Default to strict mode: do not invent unknown fields, and label likely or common fields as needing confirmation.
4. If context is unclear, ask or provide clearly labelled context-specific versions. Do not silently assume one.
5. Apply the guardrails, patterns, warnings, and output rules in [the macro reference](./references/macro-assistant-reference.md).
6. Keep the answer concise but include the key assumptions, risks, and testing advice.

## Output Expectations

- For creation: provide the macro, context, short explanation, and any field or testing warnings.
- For debugging: state whether it is valid, risky, or invalid; list the issues; provide a fixed version; explain the changes.
- For complex Email HTML: clearly warn that Preview or Macro Evaluator testing is required, label the result as cautious or untested, and prefer keeping complex logic in Workflow MCR.

## Notes

- Prefer the known or common patterns in the reference before falling back to placeholders.
- When a likely field is suggested, tell the user to confirm it with Ctrl + Shift + R or Data Field Map.
