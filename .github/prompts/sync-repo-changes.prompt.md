---
description: "Use when a workflow rule, guardrail, or behavioral change has been made to any agent file and needs to be propagated to all relevant files and committed to the GitHub repo."
---

A change has been made (or a change is described below). Propagate it to every file that needs it, then commit and push to https://github.com/James-E70/VS-Code-Native-Support-Agent.

## Files to check and update

Before committing, review each of the following locations and update any that do not yet reflect the change. Not every file will need updating for every change — use judgment about relevance, but err on the side of inclusion for guardrails and workflow rules that affect response quality.

| File | Purpose |
|---|---|
| `SPSS PROMPT.md` | Primary active prompt. The authoritative source for investigation, guardrail, and workflow rules used during live support runs. |
| `.github/copilot-instructions.md` | Auto-loaded workspace instructions. Must always mirror `SPSS PROMPT.md` exactly — update this file whenever SPSS PROMPT.md changes so that agent sessions pick up the latest rules without manual file attachment. |
| `.github/skills/wisetech-support-response/SKILL.md` | Skill file loaded when generating client-facing eRequest responses. Must stay aligned with SPSS PROMPT.md on response style, footer format, and workflow gates. |
| `support-agent-workflow.md` | Repository-tracked workflow reference. Keep in sync with SPSS PROMPT.md for any rule that affects investigation quality, evidence handling, or response production. |
| `.github/prompts/support-agent.prompt.md` | Reusable prompt for starting support investigations. Update if the change affects how investigations are initiated or how attachments/incidents are handled. |
| `/memories/support-agent-preferences.md` (user memory) | Persistent cross-workspace agent preferences. Update if the change is a behavioral rule that should apply in all workspaces and conversations, not just this repo. |
| `/memories/repo/support-agent-workflow.md` (repo memory) | Repo-scoped memory facts. Update if the change is a repo-specific fact, tool behavior note, or environment-specific rule. |

## Steps

1. Read the current content of each file listed above.
2. For each file that is missing the change, apply the update. Keep the wording consistent across files — the same rule should not mean different things in different files.
3. Run `git status` to confirm which files have been modified.
4. Stage all modified files with `git add`.
5. Commit with a message that describes what changed and why (include the root cause or incident reference if applicable).
6. Push to `origin main`.
7. Report the commit hash and a summary of which files were updated and which were already current.

## Notes

- Do not commit response files such as `CS########_response.txt` or `CS########_response.b64` — these are working artefacts, not repo content.
- Do not commit `.gitignore`-excluded files.
- If the change is to user memory only (not a repo file), note that it is local and not pushed to GitHub.
- If a file cannot be updated because it does not exist or the path is unclear, state that explicitly rather than silently skipping it.
