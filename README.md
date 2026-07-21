# VS-Code-Native-Support-Agent

VS Code native support-agent configuration for investigating CargoWise eRequests and drafting client-facing responses or internal escalation notes from current incident evidence.

## Purpose

This repository packages the prompt files, skills, instructions, and workspace configuration used to run a support-focused Copilot agent inside VS Code.

The agent is designed to:
- retrieve incident details and attached evidence directly from ediProd
- investigate the issue to the most solved outcome support can reasonably achieve before escalating
- uses validated fixes, validated workarounds, validated configuration corrections, or one precise next diagnostic step
- generate a client-facing response saved as a `.txt` file in the workspace folder, ready for review before sending
- draft an internal escalation note (when prompted) when the incident requires escalation or a documented internal handoff

## Core Behaviour

The agent is instructed to:
- treat every eRequest as a fresh investigation based on the current incident description, latest client updates, and attached evidence
- check for duplicate open incidents from the same organisation before beginning a new investigation
- search for related Work Items in the WiseTech product backlog as part of every investigation — if a matching open or closed Work Item is found, the agent incorporates its status, fixed build, and any documented workaround directly into the response
- avoid repeating checks already proven by the latest evidence
- prefer exact error text, Workflow and Tracking events, exported XML, logs, and other machine-verifiable artefacts over generic screenshots when they are more decisive
- inspect direct image attachments visible in the incident's eDocs
- treat converted spreadsheet text and tables as primary evidence before relying on linked fallback images
- treat DOCX or PDF conversions that expose `ediprod:///docs/.../images/...` links as reviewable document-image output and open those linked images before classifying the source file as unparsed
- treat any skipped, unsupported, unreadable, or unparsed attachment as an incomplete evidence review and surface `FILES COULD NOT BE PARSED: ...` in the chat summary
- never provide unverified UI paths, fields, or assumptions
- automatically route macro-related incidents to the `wisetech-macro-assistant` skill before analysis begins — this skill is included in this repository under `.github/skills/wisetech-macro-assistant/` and covers macro creation, debugging, DocBuilder expressions, and macros for Workflow and Emails
- Automatically use the Kibana GlobalSearch investigation workflow (`.github/guides/kibana-performance-investigation.md`) when a WiseCloud-hosted incident involves SQL lock timeouts, force-close events on save, slow performance across multiple users, or system-wide blocking — the shareable prompt at `shareable-prompts/Kibana_Performance_Investigation.prompt.md` provides a ready-to-use entry point for this workflow
- verify UI paths against WTA user documentation before including them in any client-facing response
- fire a hard VERSION NUMBER GATE when the next investigation step depends on knowing the client's CargoWise build
- fire a hard HOSTING GATE when the next investigation step differs between self-hosted and WiseCloud-hosted environments

## Output Standard

The generated response:
- starts with `Hi <Contact First Name>,` (with Contact Name replaced by the first name of the client contact reporting the eRequest)
- uses business-appropriate language with clear paragraph spacing
- states product behavior directly from current evidence — no attribution phrasing such as "According to WTA..."
- includes inline URL citations for every WTA article, Update Note, how-to, or guide referenced in the body

Footer content:
- confidence rating (`Confidence: X/5`)
- AI disclaimer
- up to 5 similar or related incidents from the last 3 years, each with a description and resolution outcome
- up to 5 relevant WiseTech Academy links when genuinely relevant

## Repository Structure

```
.github/
  copilot-instructions.md          # Auto-loaded workspace instructions — mirrors SPSS PROMPT.md exactly
  guides/
    kibana-performance-investigation.md   # Technical workflow for Kibana GlobalSearch investigations
  instructions/
    macro-routing.instructions.md  # Auto-routing rule: loads wisetech-macro-assistant skill for macro incidents
  prompts/
    support-agent.prompt.md        # Reusable prompt for starting support investigations
    sync-repo-changes.prompt.md    # Prompt for propagating rule changes across all relevant files
  skills/
    wisetech-macro-assistant/      # Skill for macro creation, debugging, DocBuilder, barcode, and label questions
    wisetech-support-response/     # Skill for generating client-facing eRequest responses
    wisetech-support-response-consolidated/  # Self-contained consolidated version of the above
shareable-prompts/
  Kibana_Performance_Investigation.prompt.md  # Shareable prompt for Kibana performance investigation workflow
  Triage_eRequest.prompt.md        # Shareable prompt for triaging eRequests
.vscode/
  settings.json                    # Local workspace settings
SPSS PROMPT.md                     # Primary active prompt — authoritative source for all investigation and guardrail rules
Prompt For Support Agent - With eDocs.md  # Legacy detailed operating prompt (retained for reference)
support-agent-workflow.md          # Repository-tracked workflow notes mirroring operating guardrails
.gitignore                         # Excludes generated response and escalation .txt artefacts from tracking
```

## How To Use In VS Code

Before you begin, you will need two things installed on your computer:

1. **VS Code** — the free code editor this agent runs inside. Download it from [https://code.visualstudio.com/download](https://code.visualstudio.com/download) and run the installer.
2. **GitHub Copilot Chat** — the AI extension for VS Code. Once VS Code is open, click the Extensions icon in the left sidebar (it looks like four squares), search for "GitHub Copilot Chat", and click Install. You will need an active GitHub Copilot licence to use it.

Once both are installed, download this repository:

3. Go to [https://github.com/James-E70/VS-Code-Native-Support-Agent](https://github.com/James-E70/VS-Code-Native-Support-Agent), click the green **Code** button, and select **Download ZIP**. Extract the ZIP to a folder on your computer.

Then open and use the agent:

4. Open VS Code, go to **File > Open Folder**, and select the folder you just extracted.
5. Open Copilot Chat — the workspace instructions in `.github/copilot-instructions.md` are loaded automatically.
6. Paste or type the incident number (e.g. `CS02407844`) into the chat. The agent will retrieve the incident and attached evidence from ediProd directly.
7. Review the investigation summary in chat, then open the drafted response `.txt` file saved to the workspace folder and review it before sending or uploading to eDocs.

To start a Kibana performance investigation for a WiseCloud SQL lock or blocking incident, use `shareable-prompts/Kibana_Performance_Investigation.prompt.md`.

To triage an eRequest, use `shareable-prompts/Triage_eRequest.prompt.md`.

If you want a standalone version of the response-generation rules without the full repository setup, the `.github/skills/wisetech-support-response-consolidated/SKILL.md` file is self-contained and can be downloaded and attached independently to any Copilot Chat session.

## Operational Notes

- Partial attachment review must not be treated as complete evidence review. If any attachment cannot be reviewed, the warning `FILES COULD NOT BE PARSED: ...` must appear in chat before the final conclusion is drafted.
- Image-only DOCX or PDF evidence may arrive as a converted markdown stub plus linked page images. Those linked `ediprod:///docs/.../images/...` files are part of the review path and must be opened in staged batches before requesting re-exported screenshots.
- The agent automatically checks whether any recommended client-facing step is feasible for the client's hosting model and access level before including it in the response — this is part of the agent's built-in behaviour and does not require any action from the user.
- Any client-facing output should be reviewed by a support specialist before sending.
- The `wisetech-macro-assistant` skill is loaded automatically via the instruction file whenever a macro-related keyword is detected in the incident title, description, or any eConversation post. It does not need to be manually invoked.

## Intended Audience

This repository is intended for colleagues who want a reusable, VS Code-native support-agent setup for CargoWise eRequest investigation and response drafting.
