# AGENTS.md

This repository is a **template**: teams clone it and continue development.
Agent behavior must optimize for long-term maintainability, reproducible onboarding, and documentation quality.

## Core goals

- **Reproducible dev**: same toolchain via Dev Containers.
- **Safe-by-default**: no accidental public exposure; Cloudflare Access preferred for internal tools.
- **Fast onboarding**: `Dev Containers: Reopen in Container` should be enough to start.
- **Template durability**: docs and config must stay accurate after each session.

## Project map

- `backend/` - NestJS API submodule (default `APP_PORT=4242`)
- `frontend/` - Next.js app submodule (`3003`)
- `.devcontainer/devcontainer.json` - editor/runtime environment
- `.devcontainer/docker-compose.yml` - local services (`db`, `backend`, `frontend`, `mailcatcher`, `adminer`, `cloudflared`)

Project map is descriptive, not rigid. In cloned projects, real structure can differ.

## Documentation policy

Treat `/docs` as source of truth for decisions and rollout.

- If assumptions change, update docs in the same session.
- Keep docs internally consistent (ports, service names, file paths, commands).
- If a file is missing expected details, add them (do not wait for a separate request).
- If conflicting statements exist, resolve them and note what changed.
- For significant changes, update relevant README/docs before finishing.

## Configuration consistency rules

When changing infra/dev setup, keep these files aligned:

- `.devcontainer/devcontainer.json`
- `.devcontainer/docker-compose.yml`
- `.devcontainer/.env.example`
- `README.md`
- `.devcontainer/README.md`

Always verify:

- port mapping consistency
- service names and startup behavior
- token/secret handling instructions
- file/path references are real

## Security baseline

- Never commit real secrets (`.env`, tunnel credentials, API keys).
- Commit only templates/examples (for example `.env.example`).
- Prefer non-root operation and minimal host mounts.
- Treat public tunnels as demo-only unless explicitly justified.

## Propose

Be proactive. In meaningful sessions, propose improvements for:

- documentation quality
- infra reliability
- dev environment DX
- security guardrails

Use concise, actionable suggestions with priorities:

- `P0` critical correctness/security
- `P1` reliability/maintainability
- `P2` DX/quality optimizations

## Template evolution rule

This repository is a starter template, not a fixed architecture.

- If folders, stack, infra, or docs structure change, update `AGENTS.md` in the same session.
- When the agent understands the actual project shape in a new cloned repo, it should adapt this file to reflect reality.
- Avoid hard assumptions tied to one stack or folder layout; keep guidance portable.
- Treat `AGENTS.md` as living context that evolves with the project.

## Session maintenance rule

If architecture, workflow, tooling, or environment conventions change:

- update `AGENTS.md` and relevant docs in the same session
- keep updates concise, factual, and template-friendly
- resolve contradictions instead of appending conflicting notes

## Write It Down - No "Mental Notes"!

- **Memory is limited** — if you want to remember something, WRITE IT TO A FILE
- "Mental notes" don't survive session restarts. Files do.
- Capture high-signal knowledge: decisions, incident learnings, recurring pitfalls, and team conventions.
- Do not document every temporary thought; prefer concise, reusable notes.
- When you learn a durable lesson -> update `AGENTS.md` or relevant file in `/docs`.
- When you make a meaningful mistake -> document prevention guidance for future sessions.

## Safety

- Don't exfiltrate private data. Ever.
- Don't run destructive commands without explicit user approval.
- Examples: `git reset --hard`, `rm -rf`, force-push on shared branches, dropping/recreating databases, destructive docker prune.

## Completion check

Before finishing substantial work, run a brief consistency check:

- docs and config still align (ports, services, commands, file paths)
- changed behavior is reflected in README and `/docs` where relevant
- no stale references to removed files or old structure
