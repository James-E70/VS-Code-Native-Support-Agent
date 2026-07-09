---
description: "Use when answering CargoWise macro questions, macro debugging, Workflow MCR, Document or DocBuilder expressions, Email HTML macro requests, field mapping, barcode macros, WhsDeliveryLabels, DocBuilder label customisation, or CargoWise macro best-practice questions. Route those requests through the wisetech-macro-assistant skill and apply its safety rules."
name: "CargoWise Macro Routing"
---

# CargoWise Macro Routing

## Auto-Invoke Rule

Before beginning any investigation or response, check whether the incident or request involves any of the following. If any match, load and apply the workspace skill at `./.github/skills/wisetech-macro-assistant/SKILL.md` FIRST before analysis begins — do not defer this until after investigation.

Trigger on any of:
- Macro creation, debugging, or explanation
- Workflow MCR conditions or triggers
- Document / DocBuilder expressions or labels
- Email HTML macro syntax
- Barcode macros (e.g. AbriBar128sBarCode, QR code macros)
- WhsDeliveryLabels or warehouse label templates
- Field token questions (e.g. `<TransportReference>`, `<JS_*>` fields)
- WorkflowItems, Event.Params, collections, or pattern matching
- Any eRequest whose title or description contains the words: macro, barcode, expression, template, DocBuilder, label, field mapping, WhsDelivery, MCR, USR, HTML template, binding member, Data Field Map, field token, Ctrl+Shift+R

## eRequest Investigation Rule

When processing a support eRequest (CS incident), read the incident title, description, and all eConversation posts (client updates within the incident thread) before starting the investigation. If the title, description, eConversation posts, or any attachment references macro syntax, barcode rendering, DocBuilder expressions, label template customisation, or the word "macro", treat it as a macro-involved incident and load the skill immediately.

Do not wait for a macro-specific user prompt. The skill must be loaded at the start of the investigation, not discovered mid-response.

Root cause: CS02383547 (July 2026) — client's re-open post included the phrase "macro names"; the trigger check only covered title/description, not eConversation posts, so the skill was not loaded.

## Mixed Requests

If the request mixes macro work with a normal support investigation, apply the macro skill to the macro portion and keep the rest of the answer aligned with the active support task.
