# CLAUDE.md

This checkout is **OpenTag** (upstream: https://github.com/CopilotKit/OpenTag), running
locally as the managed Slack Channel `copilotkit-channels` through CopilotKit Intelligence.

The upstream agent instructions apply in full. Read them first:

@AGENTS.md

This file adds only what is specific to this checkout.

## This instance

| What | Value |
| --- | --- |
| Intelligence project | `copilotkit-channels` (id `6702`), in `.copilotkit/project.json` |
| Channel name | `copilotkit-channels`: `.copilotkit/channels.json` and `INTELLIGENCE_CHANNEL_NAME` must match exactly |
| Provider | Slack only. No Teams adapter is declared. |
| Runtime | `pnpm runtime` (`tsx server.ts`), port 3000, base path `/api/copilotkit` |
| Agent | `pnpm agent` (LangGraph over AG-UI, Python via `uv`), port 8123 |
| Package managers | pnpm, pinned by `packageManager` in `package.json`; `uv` for `agent/` |

Ports 8001 and 8080 are used by other services on this machine. Keep the runtime on 3000
and the agent on 8123.

## Environment and secrets

- A single root `.env` is loaded by both the runtime (`dotenv/config`) and the agent
  (`agent/agent.py`). It is gitignored. Never commit it, and never `cat`, echo, or log it.
  Check variables by name only, for example `grep -c '^NAME=.' .env`.
- Required: `OPENAI_API_KEY`, `INTELLIGENCE_API_KEY`, `INTELLIGENCE_CHANNEL_NAME`, `AGENT_URL`.
  `app/env.ts` is the contract.
- `CPK_INTELLIGENCE_API_KEY` is the name `copilotkit project select` writes. OpenTag reads
  `INTELLIGENCE_API_KEY`, which holds the same value. If the key is re-provisioned,
  update both.
- The `INTELLIGENCE_CHANNEL_COPILOTKIT_CHANNELS_SLACK_*` lines were only needed for
  `copilotkit channels add`. Intelligence now holds the Slack credentials, and AGENTS.md
  says no platform credential belongs here, so deleting those lines is safe.
- Don't reuse the `open-tag` Channel name. It would race another runtime for deliveries.

## Running and verifying

- Prefer `pnpm agent` and `pnpm runtime` over `pnpm dev`. The `predev` hook downloads
  Playwright Chromium, which this setup has not needed.
- Restart the runtime after changing Channel wiring, handlers, or the agent. It does not
  hot-reload.
- Status: `npx copilotkit@latest channels status --json` should show
  `adapters.slack: "attached"`, `server: "present"`, and `error: null`.
- `/api/copilotkit/info` returning 200 is not proof that Slack works. The proof is a real
  `@copilotkit-channels` mention in an invited channel that gets a reply. The agent's
  access log shows a `POST /` for each run.
- To see Channel lifecycle logs, restart the runtime with `LOG_LEVEL=debug`.
- Before calling work done, run the checks listed in AGENTS.md ("Verify before claiming done")
  and report only the ones you actually ran.

## Local files that aren't upstream

- `.copilotkit/` holds the CopilotKit CLI's project and Channel records, plus
  `artifacts/copilotkit-channels/` (the Slack manifest and its create-app URL). It is
  untracked.
- `.gitignore` gained `.env.*` and `!.env.example`, added by the CopilotKit CLI.
