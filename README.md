# eRequest Investigation Agent for VS Code

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
- fire a mandatory Post-Attachment Root Cause Gate after all attachments are reviewed and before any hypothesis-forming WTA search: searches for a matching WI using any exact error string reported on the eRequest, as well as the functional failure mode described in plain language, checks cross-org incident patterns via the WI's attached incidents or a keyword filter search, and writes the results in chat before any investigation direction is adopted
- when a WI or similar incident match is returned by the root cause gate, treat it as a hypothesis — not a confirmed diagnosis: verify any specific error string, failure output (log message, error code, behavioral signature), or failure scenario described in the WI is explicitly present or clearly reflected in the current evidence before anchoring on it; if the evidence would not independently lead to the same conclusion without the WI match, verification is incomplete
- avoid repeating checks already proven by the latest evidence
- prefer exact error text, Workflow and Tracking events, exported XML, logs, and other machine-verifiable artefacts over generic screenshots when they are more decisive
- inspect direct image attachments visible in the incident's eDocs
- treat converted spreadsheet text and tables as primary evidence before relying on linked fallback images
- treat DOCX or PDF conversions that expose `ediprod:///docs/.../images/...` links as reviewable document-image output and open those linked images before classifying the source file as unparsed
- treat any skipped, unsupported, unreadable, or unparsed attachment as an incomplete evidence review and surface `FILES COULD NOT BE PARSED: ...` in the chat summary
- never provide unverified UI paths, fields, or assumptions
- automatically route macro-related incidents to the `wisetech-macro-assistant` skill before analysis begins — this skill is included in this repository under `.github/skills/wisetech-macro-assistant/` and covers macro creation, debugging, DocBuilder expressions, and macros for Workflow and Emails
- Automatically use the Kibana GlobalSearch investigation workflow (`.github/guides/kibana-performance-investigation.md`) when a WiseCloud-hosted incident involves SQL lock timeouts, force-close events on save, slow performance across multiple users, or system-wide blocking — the prompt at `.github/prompts/Kibana_Performance_Investigation.prompt.md` provides a ready-to-use entry point for this workflow
- verify UI paths against WTA user documentation before including them in any client-facing response
- fire a hard VERSION NUMBER GATE when the next investigation step depends on knowing the client's CargoWise build
- fire a hard HOSTING GATE when the next investigation step differs between self-hosted and WiseCloud-hosted environments
- **External Website Check functionality:** automatically fetch and verify any fact that is available on an external website, is date-sensitive or version-specific, and where getting it wrong would change the investigation conclusion — this includes customs authority tariff and declaration rules, tax authority rulings, third-party software navigation paths, and carrier or portal service status; if the URL cannot be retrieved, the agent flags it as an evidence gap rather than proceeding on assumption

## Output Standard

The generated response:
- starts with `Hi <Contact First Name>,` (with Contact First Name replaced by the first name of the most recent active client correspondent in the eConversation thread — falling back to the original reporter if the correspondent cannot be identified; parse the author identifier by stripping the domain, splitting on the first period, and capitalizing)
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
    Kibana_Performance_Investigation.prompt.md  # Prompt for Kibana performance investigation workflow
    Triage_eRequest.prompt.md      # Prompt for triaging eRequests
  skills/
    wisetech-macro-assistant/      # Skill for macro creation, debugging, DocBuilder, barcode, and label questions
    wisetech-support-response-consolidated/  # Self-contained skill for generating client-facing eRequest responses (with embedded macro rules)
.vscode/
  settings.json                    # Local workspace settings
SPSS PROMPT.md                     # Primary active prompt — authoritative source for all investigation and guardrail rules
support-agent-workflow.md          # Repository-tracked workflow notes mirroring operating guardrails
.gitignore                         # Excludes generated response and escalation .txt artefacts from tracking
```

## How To Use In VS Code

Before you begin, you will need three things installed or configured:

1. **VS Code** — the free code editor this agent runs inside. Download it from [https://code.visualstudio.com/download](https://code.visualstudio.com/download) and run the installer.
2. **GitHub Copilot Chat** — the AI extension for VS Code. Once VS Code is open, click the Extensions icon in the left sidebar (it looks like four squares), search for "GitHub Copilot Chat", and click Install. You will need an active GitHub Copilot licence to use it.
3. **MCP Servers** — the agent relies on MCP (Model Context Protocol) servers to access ediProd incidents and the WiseTech Knowledge Base. Follow the setup steps at [https://github.com/WiseTechGlobal/WTG.AI.IL.Product/blob/main/setup-vscode-mcp/wtg-vscode-mcp/setting-up-mcp-vscode.md](https://github.com/WiseTechGlobal/WTG.AI.IL.Product/blob/main/setup-vscode-mcp/wtg-vscode-mcp/setting-up-mcp-vscode.md) to configure these before use.

Once all three are set up, download this repository:

4. Go to [https://github.com/James-E70/eRequest-Investigation-Agent-for-VS-Code](https://github.com/James-E70/eRequest-Investigation-Agent-for-VS-Code), click the green **Code** button, and select **Download ZIP**. Extract the ZIP to a folder on your computer.

Then open and use the agent:

5. Open VS Code, go to **File > Open Folder**, and select the folder you just extracted.
6. Open Copilot Chat — the workspace instructions in `.github/copilot-instructions.md` are loaded automatically.
7. Paste or type the incident number (e.g. `CS02407844`) into the chat and press Enter/Send. The agent will retrieve the incident and attached evidence from ediProd directly.
8. Review the investigation summary in chat, then open the drafted response `.txt` file saved to the workspace folder and review it before copying and sending to the client.

To start a Kibana performance investigation for a WiseCloud SQL lock or blocking incident, type `/` in Copilot Chat and select **Kibana-Performance-Investigation**. Note that the Kibana investigation workflow is also invoked automatically during a standard investigation run when the agent determines it is warranted — the `/Kibana-Performance-Investigation` command is simply a manual entry point if you want to run it directly.

To triage an eRequest, type `/` in Copilot Chat and select **Triage-eRequest**.

To draft an internal escalation note after completing an investigation, type `/` in Copilot Chat and select **Draft-Escalation-Note**. The agent will generate a structured internal summary of the issue, evidence, and escalation rationale, and save it as a `.txt` file in the workspace folder, which you can then copy and paste into the eRequest as an Internal Note or note against a Workflow Task.

To remove old response and escalation `.txt` files from the workspace folder, type `/` in Copilot Chat and select **Clean-Incident-TXT-Files**. This is a useful end-of-day cleanup command to clear out the workspace folder of that day's generated response / escalation note `.txt` files.

If you want a standalone version of the response-generation rules without the full repository setup, the `.github/skills/wisetech-support-response-consolidated/SKILL.md` file is self-contained and can be downloaded and attached independently to any Copilot Chat session.

## Operational Notes

- Partial attachment review must not be treated as complete evidence review. If any attachment cannot be reviewed, the warning `FILES COULD NOT BE PARSED: ...` must appear in chat before the final conclusion is drafted.
- Image-only DOCX or PDF evidence may arrive as a converted markdown stub plus linked page images. Those linked `ediprod:///docs/.../images/...` files are part of the review path and must be opened in staged batches before requesting re-exported screenshots.
- The agent automatically checks whether any recommended client-facing step is feasible for the client's hosting model and access level before including it in the response — this is part of the agent's built-in behaviour and does not require any action from the user.
- Any client-facing output should be reviewed by a support specialist before sending.
- The `wisetech-macro-assistant` skill is loaded automatically via the instruction file whenever a macro-related keyword is detected in the incident title, description, or any eConversation post. It does not need to be manually invoked.
- WI and similar incident matches from the Post-Attachment Root Cause Gate are hypotheses, not confirmed diagnoses — surface-level symptom similarity is not sufficient. The agent verifies that any error string, failure output, or failure scenario in the WI is present or clearly reflected in the current incident's evidence before adopting it as the investigation direction.

## Intended Audience

This repository is intended for colleagues who want a reusable, VS Code-native support-agent setup for CargoWise eRequest investigation and response drafting.
