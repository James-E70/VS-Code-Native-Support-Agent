# Support Agent Workflow Notes

This file contains repository-tracked workflow guardrails that should remain aligned with the local support-agent operating rules.

## Attachment Review Guardrails

- Before drafting any client-facing response, every eDocs file currently listed on the incident must have had a read attempt via `mcp_ediprod_read-file`. This applies to all files in the attached documents table — not only those added in the most recent client update. When a client adds new attachments, read them in addition to, not instead of, all previously existing attachments.
- Never ask a client to check, confirm, or provide information that is already visible in any attachment currently on the incident, whether or not that attachment was added recently.
- For direct image attachments already visible in the chat context, inspect the image itself before deciding whether the evidence is readable.
- When a client adds a new direct image attachment during the investigation, treat that attachment as a blocking evidence gate only for conclusions that depend on it.
- Do not draft, upload, or finalize a client-facing response that relies on a newly attached PNG, JPG, JPEG, GIF, or WEBP until that image has been directly reviewed or a concrete tool/access blocker has been explicitly stated.
- Do not infer what a screenshot shows from the client's wording, incident metadata, OCR absence, attachment filename, or similar-incident precedent when the image itself has not been directly reviewed.
- If a direct image attachment cannot be opened with available tools, state that limitation explicitly and treat the screenshot evidence as unresolved rather than substituting a hypothesis.
- Do not treat missing OCR text, missing extracted text, or missing parser output as sufficient reason to mark a direct image attachment as unparsed.
- For converted spreadsheets such as XLS, XLSX, or XLSM, treat worksheet text, tables, and markdown output as the primary evidence source before relying on linked fallback images.
- If a DOCX or PDF conversion returns markdown image links under `ediprod:///docs/.../images/...`, treat that output as a usable extraction path and open those linked images before deciding the document could not be reviewed.
- Review linked document images in small staged batches and summarize findings between batches instead of pulling every page at once.
- If a converted Office attachment exposes fallback image links and one linked image appears invalid or unreadable as an image resource, skip that image, continue with the remaining parsed content, and only treat the file as incomplete if the decisive evidence exists solely in the broken image.
- When a DOCX or PDF yields only linked page images, treat those images as the primary evidence source for that document during the investigation.
- Transcribe or summarize the relevant visible text from reviewed page images into the working notes or chat summary so image-only documents remain usable without native extracted text.
- For long multi-page documents, prioritize the pages most likely to contain decisive evidence first, then continue in staged batches only as needed.
- If linked page images exist but the text is still unreadable, request only the minimum better-format follow-up needed, such as higher-resolution page exports or a searchable OCR copy.
- Use the `FILES COULD NOT BE PARSED: ...` warning only after direct image review or document-image review has been attempted and still failed or remained unavailable.
- Unresolved attachments block only the conclusions that depend on them; they do not require the entire investigation to stop when the remaining reviewed evidence independently supports another conclusion.

## Verified Action Guardrails

- Do not ask a client to perform any step unless the step itself is standard for the described context or otherwise verified to be possible for that client context.
- Verify role, access, hosting, and tool prerequisites before suggesting an action.
- For an exact CargoWise support incident number in the form `CS########`, treat the incident number itself as the first retrieval anchor.
- Before concluding that the exact incident reader is unavailable for a known `CS########` incident, first try to surface the deferred issue-management activation tool with the literal query `activate_workitem_and_issue_management deferred tool`.
- If that activation tool is surfaced, call the deferred tool named `activate_workitem_and_issue_management` before doing anything else with exact incident retrieval.
- Do not treat a surfaced activation tool as sufficient on its own. The required step is the actual `activate_workitem_and_issue_management` tool call.
- After a successful `activate_workitem_and_issue_management` call, immediately try a direct `mcp_ediprod_get-job-details` call with the exact `CS########` job number before doing any further `tool_search` for job-details.
- Do not run another `tool_search` for `mcp_ediprod_get-job-details` after a successful `activate_workitem_and_issue_management` call until you have first attempted that direct tool call.
- The first retrieval path for an exact `CS########` incident is `mcp_ediprod_get-job-details` with that exact job number. If the tool is not active yet, do one targeted tool search for the exact job-details tool, then call it immediately.
- When tool search is required for that exact path, the first query should be the literal string `mcp ediprod get job details incident`.
- If that exact literal query does not surface `mcp_ediprod_get-job-details`, do at most one alternate minimal query `mcp ediprod get job details`, then stop searching and move according to the fallback rules.
- Do not declare the exact incident tool unavailable until the `activate_workitem_and_issue_management` call has either succeeded or clearly remained unavailable, and then the literal `mcp ediprod get job details incident` query and the single alternate `mcp ediprod get job details` query have both been tried.
- If `activate_workitem_and_issue_management` has succeeded, a direct `mcp_ediprod_get-job-details` call is the next required step and should happen before concluding that the exact tool is unavailable.
- If the first exact job-details call fails with a transport or endpoint error, retry the same call once before switching to any broader fallback.
- When exact job details are returned, use that payload and its attachment URLs as the evidence anchor, then read the attachments with `mcp_ediprod_read-file` before attempting weaker lookup paths.
- Do not use `mcp_ediprod_get-product-info` while still attempting to surface or execute the exact job-details path for a known `CS########` incident.
- `mcp_ediprod_get-product-info` is only justified after the exact path has failed and you have explicitly chosen the `mcp_ediprod_filter-incidents` fallback that requires valid product keys.
- Only fall back to incident filters, queue listings, or browser or webpage access after the exact job-details path has been tried and either remained unavailable or failed after one retry.
- A miss in `mcp_ediprod_filter-incidents`, a queue export, or a browser-launch path is not enough to conclude that the incident is unavailable when the exact job-details path has not yet been tried.
- If a step depends on access to a browser, desktop shell, terminal server, local file system, published app, or OS utility, do not assume that capability exists.
- Either verify that capability from authoritative documentation or current evidence, or phrase that step conditionally with a validated fallback; ask a confirming question only when that dependency is necessary for the next step.
- When the client is WiseCloud-hosted, never assume they can open a browser or Windows shell inside the active CargoWise session.
- Treat in-session browser or shell access as unavailable unless current evidence or authoritative documentation confirms otherwise.

## Response Draft Guardrails

- Before finalising recommended next steps, do a consistency pass: if a finding has already been stated as confirmed in the response body (e.g. "the events log does not contain X, which confirms Y"), the next steps must not ask the client to check for that same thing. Next steps must begin from where the confirmed finding ends, not repeat the verification that has already been done.
- Before finalising if/then next steps, enumerate all logical states the client could be in that explain the confirmed finding. For each state that has a known self-service fix or documented resolution path, provide that fix directly with the relevant guide link — do not default to requesting more evidence when the fix is already known. Only ask for evidence in the branch where the resolution genuinely depends on information that is not yet available.

## Defect Classification Guardrails

- Do not classify an issue from the nearest similar incident alone when the product behavior may have changed since that incident was closed.
- Treat delivered update notes, delivered work items, and newer authoritative records as higher priority than older incident closure reasons when determining the current product state.
- If an older incident was closed as a feature request or expected behavior, but a later delivered update note shows that behavior was implemented, reframe the current issue as either an upgrade gap or a regression depending on the client build.
- Before calling an issue a known defect, feature request, or expected limitation, check whether version or release boundaries are the real discriminator and state that boundary explicitly.

## Usage

- Keep this file aligned with the working support-agent prompt and any repository-scoped workflow memory used during investigations.
- If a workflow rule materially changes support behaviour, update this file in the same change set so the repository remains the source of truth.
- If fresh client evidence arrives after a draft conclusion has been formed, reopen the evidence review from that attachment first and do not rely on the earlier draft until the new evidence is checked.
- Before any final user-facing chat close-out, perform a completion check covering: attachments reviewed, attachments unresolved, whether any conclusion depends on unresolved evidence, and whether the ALL CAPS parse-warning line must be included.
- Treat the final user-facing chat close-out as the required chat summary for attachment-review rules rather than as a separate delivery/status note.
- If any attachment review failed or remained unavailable, the final user-facing chat close-out must include exactly one line using this format: `FILES COULD NOT BE PARSED: <comma-separated file names>`.

## Response Footer Formatting

- The WiseTech Academy footer section must be headed exactly: `Relevant WiseTech Academy links:`.
- Each academy item must include both the module title and URL on the same line using this format: `- <module title> - <url>`.
- Do not output bare wisetechacademy.com URLs without titles.
- If no genuinely relevant academy content is available, state that explicitly under the same heading instead of listing generic links.

## Client-Facing Wording

- In client-facing responses, present supported product behavior as a direct statement of how the system works based on the current evidence and authoritative sources.
- Do not use hedging phrases such as `I am not aware of`, `I believe`, `it seems`, or similar wording for evidence-backed conclusions. When the evidence is sufficient, state the conclusion directly. If scope needs to be bounded, use wording such as `Based on the current functionality, ...`.
- Do not phrase client-facing conclusions as being confirmed by similar incidents, prior cases, or support precedents.
- Similar incidents may still be listed in the required footer, but they should not be used as the stated basis for the main response wording.
- When citing any Update Note or WiseTech Academy content in the body of a client-facing response, always include the direct link inline in the same sentence using this format: `<title or description> - <url>`.
- Final wording check: before sending any client-facing response, scan once for habit-language hedges and replace them with direct evidence-based wording.
