# Support Agent Workflow Notes

This file contains repository-tracked workflow guardrails that should remain aligned with the local support-agent operating rules.

## Verified Action Guardrails

- Do not ask a client to perform any step unless the step itself is verified to be possible for that client context.
- Verify role, access, hosting, and tool prerequisites before suggesting an action.
- If a step depends on access to a browser, desktop shell, terminal server, local file system, published app, or OS utility, do not assume that capability exists.
- Either verify that capability from authoritative documentation or current evidence, or phrase the step conditionally with a validated fallback.
- When the client is WiseCloud-hosted, never assume they can open a browser or Windows shell inside the active CargoWise session.
- Treat in-session browser or shell access as unavailable unless current evidence or authoritative documentation confirms otherwise.

## Usage

- Keep this file aligned with the working support-agent prompt and any repository-scoped workflow memory used during investigations.
- If a workflow rule materially changes support behaviour, update this file in the same change set so the repository remains the source of truth.
