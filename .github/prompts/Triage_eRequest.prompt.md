---
name: "Triage eRequest"
description: "Review the active eRequest in the current session and recommend the most appropriate Product Area, Section, and Criticality categorisation."
argument-hint: "Optional: add any triage context or constraints"
agent: "agent"
---
Review the eRequest currently being worked on in this session and recommend how it should be triaged.

Requirements:
- Identify the active incident or eRequest from the current session context, workspace files, or attached evidence.
- If the active incident cannot be identified with confidence, stop and ask only for the incident number.
- Review the available incident text, latest client updates, and attached evidence before recommending any categorisation.
- Output the final recommendation in exactly this four-line format:
- `Product Area: <value> - <short reason>`
- `Section: <value> - <short reason>`
- `Criticality: <value> - <short reason>`
- `Confidence: <0-5> - <short overall reason>`
- Do not add any extra headings, bullets, preamble, or closing text around those four lines.
- Keep the reasoning support-internal, direct, and concise.
- If the evidence is incomplete or ambiguous, say so explicitly and give the best current recommendation with the uncertainty noted.
- Do not upload anything to eDocs and do not create `.txt` or `.b64` files unless the user explicitly asks for that separately.

If the user includes extra prompt arguments, incorporate them when they do not conflict with the triage goal.
