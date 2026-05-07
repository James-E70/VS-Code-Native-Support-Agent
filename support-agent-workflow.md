# Support Agent Workflow Notes

This file contains repository-tracked workflow guardrails that should remain aligned with the local support-agent operating rules.

## Attachment Review Guardrails

- For direct image attachments already visible in the chat context, inspect the image itself before deciding whether the evidence is readable.
- When a client adds a new direct image attachment during the investigation, treat that attachment as a blocking evidence gate for any conclusion that depends on it.
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

## Verified Action Guardrails

- Do not ask a client to perform any step unless the step itself is verified to be possible for that client context.
- Verify role, access, hosting, and tool prerequisites before suggesting an action.
- If a step depends on access to a browser, desktop shell, terminal server, local file system, published app, or OS utility, do not assume that capability exists.
- Either verify that capability from authoritative documentation or current evidence, or phrase the step conditionally with a validated fallback.
- When the client is WiseCloud-hosted, never assume they can open a browser or Windows shell inside the active CargoWise session.
- Treat in-session browser or shell access as unavailable unless current evidence or authoritative documentation confirms otherwise.

## Defect Classification Guardrails

- Do not classify an issue from the nearest similar incident alone when the product behavior may have changed since that incident was closed.
- Treat delivered update notes, delivered work items, and newer authoritative records as higher priority than older incident closure reasons when determining the current product state.
- If an older incident was closed as a feature request or expected behavior, but a later delivered update note shows that behavior was implemented, reframe the current issue as either an upgrade gap or a regression depending on the client build.
- Before calling an issue a known defect, feature request, or expected limitation, check whether version or release boundaries are the real discriminator and state that boundary explicitly.

## Usage

- Keep this file aligned with the working support-agent prompt and any repository-scoped workflow memory used during investigations.
- If a workflow rule materially changes support behaviour, update this file in the same change set so the repository remains the source of truth.
- If fresh client evidence arrives after a draft conclusion has been formed, reopen the evidence review from that attachment first and do not rely on the earlier draft until the new evidence is checked.

## Response Footer Formatting

- The WiseTech Academy footer section must be headed exactly: `Relevant WiseTech Academy links:`.
- Each academy item must include both the module title and URL on the same line using this format: `- <module title> - <url>`.
- Do not output bare wisetechacademy.com URLs without titles.
- If no genuinely relevant academy content is available, state that explicitly under the same heading instead of listing generic links.

## Client-Facing Wording

- In client-facing responses, present supported product behavior as a direct statement of how the system works based on the current evidence and authoritative sources.
- Do not phrase client-facing conclusions as being confirmed by similar incidents, prior cases, or support precedents.
- Similar incidents may still be listed in the required footer, but they should not be used as the stated basis for the main response wording.
- When citing any Update Note or WiseTech Academy content in the body of a client-facing response, always include the direct link inline in the same sentence using this format: `<title or description> - <url>`.
