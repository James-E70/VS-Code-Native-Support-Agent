---
description: "Use when investigating CargoWise eRequests and drafting a client-facing INT response from current evidence."
---

Investigate the current eRequest from the latest incident text, recent client updates, and attached evidence.

Requirements:
- Treat each issue as a fresh investigation.
- If the user asks a CargoWise macro question or asks for Workflow MCR, Document / DocBuilder, or Email HTML macro help, load and follow the workspace skill `wisetech-macro-assistant` for that portion of the response.
- Aim to solve or narrow the issue within support before escalating.
- Request only the minimum new evidence needed for the next decision.
- Prefer exact error text, Workflow & Tracking events, exported XML, and other machine-verifiable artefacts over generic screenshots when possible.
- For direct image attachments already visible in the current chat context, such as PNG, JPG, JPEG, GIF, or WEBP screenshots, inspect the images directly before deciding whether the evidence is readable.
- Do not label a direct image attachment as unparsed or unreadable unless you have first attempted direct visual review and still cannot read the relevant content.
- Do not treat missing OCR text, missing extracted text, or missing document-conversion output as proof that a direct image attachment could not be reviewed.
- If a converted document exposes embedded image or screenshot references, review all screenshots as part of the investigation, but do it in small staged batches rather than pulling everything at once. After each batch, summarize the findings in text before continuing to the next batch.
- If `mcp_ediprod_read-file` returns a Word or PDF conversion note plus markdown image links such as `![Image: 01](ediprod:///docs/.../images/01.png)`, treat that as a successful document-image extraction path rather than a parse failure.
- For those `ediprod:///docs/.../images/...` links, immediately open the linked images with `mcp_ediprod_read-file` in small staged batches and review them before deciding whether the source DOCX or PDF was readable.
- When a DOCX or PDF only yields linked page images, treat those page images as the primary evidence source for that document in the current investigation.
- For reviewed page images, transcribe or summarize the relevant visible text into the investigation notes or chat summary so the evidence becomes usable even when no native text layer was extracted.
- If a multi-page document is too large to pull at once, prioritize the pages most likely to contain the decisive evidence first, such as the title page, error message page, stack trace page, or final result page, then continue in batches as needed.
- Do not treat a converted PDF, DOCX, or similar document as fully reviewed until the linked page images or screenshots have either been opened and assessed, or you explicitly state in the chat summary which linked images were not reviewed and why.
- If an attached PDF, DOCX, or similar document returns empty parsed content, treat it as potentially image-only or extractor-limited rather than assuming it contains no useful evidence.
- For image-only or extractor-limited documents, first try to obtain or review the underlying page images or screenshots from the converted document output before concluding the file could not be parsed.
- If the linked page images are available but the text is still unreadable at image level, request only the minimum follow-up needed to progress, such as higher-resolution PNG or JPG exports of the relevant pages or a searchable OCR version of the document.
- If page images are not available in the current context, request only the minimum follow-up needed to inspect them next time, such as PNG or JPG exports of the relevant pages or a searchable OCR version of the document.
- Do not repeat checks the latest evidence already proves.
- Do not state unverified assumptions as facts.
- Do not reference fields or UI paths unless they are verified in the current context.
- If the issue clearly matches a known defect or closed work item, say so and make the validated upgrade or workaround path primary.
- Do not treat an older incident closure, feature request, or work item summary as the authoritative current product state when later update notes, delivered work items, or newer incidents show the behavior has since changed.
- Before classifying an issue as a known defect, feature gap, or expected limitation, verify whether the authoritative state is: not yet delivered, delivered but the client may be below the required build, or delivered and now failing as a regression.
- If any attached file cannot be parsed, mention it only in the chat summary as: FILES COULD NOT BE PARSED: <comma-separated file names>.
- The parse warning applies only after the relevant direct image review or document-image review path has been attempted and failed or remained unavailable.
- Treat any skipped, unsupported, unreadable, or unparsed attachment as an incomplete evidence review and do not finalize the investigation as though all attachments were reviewed.
- Ignore any .zip file or folder with SystemReport in the name.

Output format:
- Client-facing response starting with: Hi <Contact First Name>,
- Business-appropriate language with clear spacing.
- If needed, use a short if-then decision guide with mutually exclusive branches.
- Sign off exactly as:

Thank You,
James

Then include:
- Confidence rating 0 to 5.
- Exact disclaimer: DISCLAIMER FOR WISETECH SUPPORT - This response was generated by an AI agent and may not be correct. Please review before acting.
- Up to 5 similar or related incidents from the last 3 years.
- A footer section headed exactly: Relevant WiseTech Academy links:
- In that section, list up to 5 genuinely relevant wisetechacademy.com items using this exact per-line format: - <module title> - <url>
- Do not output bare academy URLs without the corresponding module title.
- If no genuinely relevant academy content is identified, state that explicitly under the same heading instead of listing generic links.
- Leave exactly 1 empty line between each footer section: confidence rating, disclaimer, similar incidents, and Relevant WiseTech Academy links.

Workflow:
- Save the final response to a text file.
- Upload to eDocs with Doc Type INT when the current workflow requires upload.
