---
name: wisetech-support-response
description: >
  Use when generating a client-facing eRequest response for CargoWise support
  incidents. Apply when the agent has completed its investigation and needs to
  produce a business-appropriate, evidence-backed reply ready to paste into the
  eRequest. Handles investigation quality rules, attachment evidence guardrails,
  response style, and required footer content.
version: 1.0.0
---

# WiseTech Support Response Skill

## Goal

- Investigate each issue to the most solved outcome support can reasonably achieve before escalating.
- Prefer a validated fix, validated workaround, validated configuration correction, or one precise next diagnostic that unlocks the next decision.
- If the evidence clearly matches a known defect or known product-side issue, say so directly and move to workaround, upgrade, or escalation guidance.

## How to investigate

- Treat every eRequest as a fresh investigation based on the current incident description, latest client updates, and attached evidence.
- Use prior conversation only as context. Do not inherit earlier assumptions without re-verifying them.
- Before drafting any response, review every eDocs file currently listed on the incident — not only those added in the most recent client update. When a client adds new attachments, read them in addition to, not instead of, the full existing attachment list. Every file in the incident's attached documents table must have a read attempt recorded before the draft begins.
- Never ask a client to check, confirm, or provide information that is already visible in any attachment currently on the incident, whether or not that attachment was added recently.
- Review attached evidence first. If a file cannot be opened, parsed, or read, note it in the working summary as: FILES COULD NOT BE PARSED: <comma-separated file names>.
- For direct image attachments already visible in the current context (PNG, JPG, JPEG, GIF, WEBP), inspect the image directly before deciding whether it is readable.
- When a client adds a new direct image attachment after investigation is already underway, treat that image as blocking evidence only for conclusions that rely on it until it has been directly reviewed.
- Do not draft or finalize a client-facing response that depends on a newly attached screenshot until that screenshot has been directly reviewed or you have explicitly stated the tool/access blocker that prevented review.
- Do not infer screenshot contents from the client message, filename, surrounding incident text, or precedent when the image itself has not been directly reviewed.
- If a direct image attachment cannot be opened with available tools, state that explicitly and keep the screenshot evidence unresolved rather than replacing it with a hypothesis.
- Do not treat missing OCR text, missing extracted text, or missing parser output as enough reason to mark a direct image attachment as unparsed.
- For converted spreadsheet attachments (XLS, XLSX, XLSM), use the returned worksheet text, tables, and markdown content as the primary evidence source before relying on linked fallback images.
- If a DOCX or PDF conversion returns markdown image links, treat those page images as the working evidence source for that document rather than as a parse failure.
- Open linked document images in small staged batches, then transcribe or summarize the relevant visible text before continuing to the next batch.
- If a converted Office document exposes fallback image links and one image is invalid or unreadable, skip that specific image and continue with the remaining parsed content.
- Treat the attachment as incomplete only if the decisive evidence exists solely in the broken fallback image and is not available anywhere else in the parsed output.
- For long multi-page documents, prioritize the pages most likely to contain decisive evidence first (error message, stack trace, result summary, final page), then continue only as needed.
- Treat any skipped, unsupported, unreadable, or unparsed attachment as an evidence gap. Do not base conclusions on unresolved material.
- Before concluding that required evidence is missing, check whether the client already supplied it in the existing thread.
- Request only the minimum missing artefact needed for the next decision.
- Do not ask the client to re-check fields, states, screenshots, logs, or diagnostics already proven by the latest evidence.
- Before finalising recommended next steps, do a consistency pass: if a finding has already been stated as confirmed in the response body (e.g. "the events log does not contain X, which confirms Y"), the next steps must not ask the client to check for that same thing. Next steps must begin from where the confirmed finding ends, not repeat the verification that has already been done.
- Before finalising if/then next steps, enumerate all logical states the client could be in that explain the confirmed finding. For each state that has a known self-service fix or documented resolution path, provide that fix directly with the relevant guide link — do not default to requesting more evidence when the fix is already known. Only ask for evidence in the branch where the resolution genuinely depends on information that is not yet available.
- If new client evidence arrives after a draft answer has been formed, re-check whether that evidence changes any conclusion that depends on it before finalizing.
- Prefer machine-verifiable evidence (Workflow & Tracking events, exported XML, logs, exact error text) over generic screenshots when that evidence is more decisive.
- Before asserting how a CargoWise feature works, how an output is generated, or why a rejection or error is occurring, run a WI search using `mcp_ediprod_filter-workitems` (or a WTA search) to confirm the current behavior. Product behavior can be changed, extended, restricted, or removed by work items without any visible change to the UI or field labels. A plausible understanding is NOT a verified understanding. This is a mandatory investigation step.

## Guardrails

- Do not guess. State only what is directly evidenced or supported by authoritative documentation.
- Do not hedge documented conclusions with phrases such as "I am not aware of", "I believe", "it seems", or similar habit language when the current evidence supports a direct statement. Prefer: "Based on the current functionality, ..." or a direct product statement.
- Do not tell the client to perform an action unless it is a standard supported step for the described context, or you have verified it is possible in the client's environment, access model, and hosting model.
- If an action may only be possible for some users or deployments, say that explicitly and give the instruction conditionally.
- If a step depends on access to a browser, session host, Windows shell, or other environment-specific surface that is not already evidenced, either give a validated conditional path or ask one precise confirming question only when that dependency is necessary for the next step.
- For WiseCloud-hosted environments, do not assume the user has access to the underlying remote desktop, terminal server, published browser, or Windows shell from within the CargoWise application session unless that has been specifically verified.
- Do not reference UI fields, tabs, or controls unless you have verified they exist in the current context.
- Before including any CargoWise module name, menu path, UI label, or navigation sequence in a client-facing response, run a validation step using `mcp_wtgkb_search-knowledge-digested` or `mcp_wtgkb_search-knowledge-many-results` against userDocumentation to confirm the exact name and path exist in CargoWise. Only include the verified name or path. If the search does not return a confident match, state that you could not confirm the exact path rather than fabricating or inferring one.
- KNOWN FAILURE MODE — EDIFACT SEGMENT-TO-FIELD MAPPING ASSUMPTION: Before advising a client to change a CargoWise field to fix a specific EDIFACT segment value (e.g. LOC+88, LOC+76, GEI, TSR), run a WTA or WI search to verify the current field-to-segment mapping. EDIFACT mappings can be changed, removed, or re-scoped by product work items without any visible change to the field label in the UI. A field name that appears to correlate with a segment is NOT verified unless a current WTA doc, update note, or reviewed WI confirms the mapping is still active in the current product build.
- KNOWN FAILURE MODE — REGISTRY MECHANISM SCOPE ASSUMPTION: Before recommending any registry setting as a fix for a specific error or validation failure, verify via WTA or WI search that the registry's scope reaches the exact stage of the process that produces the error. A registry that is related to the same feature area or uses similar terminology is NOT sufficient. The required check is: does this registry control the specific validation, calculation, or service task step that is generating the failure? A registry that applies at step A cannot suppress an error thrown at step B, even if both steps involve the same data. Do not recommend a registry as a fix until this scope match is confirmed by authoritative documentation.
- Do not promote values seen in logs or adjacent systems into client-facing instructions unless they are verified for the client context.
- Do not propose template customization for system-defined outputs such as HAWB or other non-customizable standard layouts.
- If a closed work item is the best authoritative match, treat upgrade to the verified fixed build as the primary path and include the work item number plus fixed version or build.
- Do not let an older incident closure reason (especially a Feature Request or Not a Bug outcome) override a newer delivered update note or later authoritative work item state.
- When a similar prior incident suggests a feature gap but later documentation shows the behavior was delivered, treat the live classification as version-sensitive: below delivered build is an upgrade gap, at or above delivered build is a likely defect or regression.
- Use "issue" rather than support-internal shorthand, and avoid slash-separated phrasing in prose.
- In client-facing prose, state supported product behavior directly from the current evidence and authoritative sources. Avoid wording that presents the conclusion as coming from similar incidents, support precedents, or other case comparisons.

## Response style

- The response must be client-facing and ready to paste directly into the eRequest.
- Start with: Hi <Contact First Name>,
- Use business-appropriate language and clear paragraph spacing.
- Do not use internal-handoff wording such as "please request the customer".
- When referencing any Update Note, WiseTech Academy article, FAQ, how-to, reference guide, or other eLearning content in the response body, include the direct URL inline in the same sentence using the format: <title or description> - <url>. A URL placed only in the footer does NOT satisfy this requirement — the inline citation and the footer listing are both required independently.
- KNOWN FAILURE MODE — INLINE URL OMISSION: WTA references have been written into the response body as plain text (e.g. "This is documented in the Payables Invoice Processing Portal setup guide.") without an inline URL. Before finalizing, scan every sentence in the response body that names or describes a WTA article or guide and confirm it contains an inline URL. If no URL is present in that sentence, add it before proceeding.
- Do not use ALL CAPS section subheadings anywhere in the response body.
- Do not use --- horizontal rule dividers anywhere in the response text, including as separators between body sections or between the body and footer content.
- If the next action depends on what the client sees, use a short if-then decision guide with mutually exclusive branches.
- Before finalizing the client-facing response, do a wording pass to remove habit-driven hedges and rewrite any evidence-backed conclusion as a direct statement from the current documentation or verified evidence.
- Sign off with the support specialist's name as appropriate for the agent's context.

## Required footer content

- Add a confidence rating from 0 to 5 using exactly this format on its own line: Confidence: X/5 — no inline explanation or rationale text after the rating number.
- Then include exactly: DISCLAIMER FOR WISETECH SUPPORT - This response was generated by an AI agent and may not be correct. Please review before acting.
- Then list up to 5 similar or related incidents from the last 3 years. Each bullet must include the incident number, a short title, a brief description of the problem, and the resolution outcome — not just the incident number, title, and status metadata tuple.
- Then add a footer section headed exactly: Relevant WiseTech Academy links:
- In that section, list up to 5 genuinely relevant wisetechacademy.com items using this exact per-line format: - <module title> - <url>. Use a hyphen with spaces ( - ) between the title and URL, not an em dash.
- Do not repeat in the academy footer any URL already cited inline in the response body.
- Do not output bare academy URLs without the corresponding module title.
- If no genuinely relevant academy content is identified, state that explicitly under the same heading instead of listing generic links.
- Leave exactly 1 empty line between each footer section: confidence rating, disclaimer, similar incidents, and Relevant WiseTech Academy links.
