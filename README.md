# Sprint Zero

A cloneable Claude Code kit that gives you a full sub-agent product team. Point it at a reference URL, answer three scoping questions, and get back a complete spec set plus a working product.

The name is a nod to agile "Sprint Zero" — the pre-sprint setup week where a team figures out what it's actually building before any real work starts. Sprint Zero the kit does that pre-sprint week for you, in one command, with Claude Code.

> This repo is in active development. Phase 1 (the skeleton you're looking at) is done. The slash commands, sub-agents, and the worked example are coming. See `plan.md` for the full roadmap.

---

## Who this is for

PMs, founders, and anyone who wants to validate an idea end-to-end without spinning up an engineering team. You bring the product judgement — the URL of something similar to what you want to build, a sense of what matters, and what doesn't. Sprint Zero handles the rest: spec generation, parallel build, browser-driven QA.

You do not need to be a developer. You do need to be able to run a few terminal commands and read what comes back.

---

## The three scope levels

Every product idea is really three products. Sprint Zero lets you choose which one you want:

1. **`clickable`** — a clickable walkthrough with fake data and no backend. Useful for pitching, flow testing, and stakeholder reviews.
2. **`working demo`** — real auth, real data, real backend. One core loop works end-to-end. This is the main output for proving the idea actually works.
3. **`pilot-ready`** — the working demo plus error handling, loading states, input validation, and enough polish to hand to 5–10 real users.

v1 is focused on `working demo` and `pilot-ready`. `clickable` is available as an escape hatch for very early-stage ideation.

---

## The stack

Sprint Zero produces one stack. Not configurable, not inferred. This is the call that makes v1 ship.

- **Frontend:** React + Vite
- **Backend:** Express (Node)
- **Database + Auth:** Supabase (Postgres + Supabase Auth)
- **Testing:** Playwright via Playwright MCP

You bring your own Supabase project — the free tier is fine. `.env.example` tells you which two values to paste. Five minutes of setup before the first run. Nothing server-side is maintained by Sprint Zero.

Out of scope for v1: multiple stacks, multi-user collaboration, deployment automation, payment integration. Those are on the v2 shortlist.

---

## Architecture, one line

URL → scoping conversation → spec set (PRD, decisions, stories, contract) → parallel build (backend + frontend) → QA with browser-driven tests → working product.

---

## Getting started

Right now the repo is the skeleton — you can clone it, read the roadmap, and see the shape of what's coming. The single-command flow (`/sprint-zero <url>`) ships in Phase 4.

When it's ready, the flow will be:

1. Clone this repo
2. Create a Supabase project and paste the URL and anon key into `.env`
3. Launch Claude Code from the repo root
4. Run `/sprint-zero <company-url>` and answer the scoping question
5. Wait for the team to finish
6. Run the app locally

Until then, the `plan.md` file shows exactly what's being built and in what order.

---

## Coming soon

Items below are tracked in `plan.md`. Nothing here is built yet.

- **Phase 2 — scoping and discovery layer:** six slash commands that turn a URL plus scoping answers into a full spec set (`docs/scope.md`, `docs/reference-brief.md`, `docs/prd.md`, `docs/decisions.md`, `docs/user-stories.md`, `docs/api-contract.md`).
- **Phase 3 — build layer:** four sub-agents (tech lead, backend engineer, frontend engineer, QA engineer) that take a spec set and produce a working product on the Express + Supabase + React + Vite stack, with Playwright-driven validation.
- **Phase 4 — kickoff orchestrator:** the single `/sprint-zero <url>` command that chains discovery and build into one flow, with named error states you can resume from.
- **Phase 5 — Mini Twenty example:** a worked example inside `examples/mini-twenty/` built by running Sprint Zero against the Twenty CRM. This is the canonical proof the kit works end-to-end.
- **Phase 6 — polish for launch:** README v2, architecture diagrams, demo GIF, `CONTRIBUTING.md`, and launch content.

---

## Status

- Phase 1 — Foundation — complete
- Phase 2 — Scoping and discovery layer — not started
- Phase 3 — Build layer — not started
- Phase 4 — Kickoff orchestrator — not started
- Phase 5 — Mini Twenty example — not started
- Phase 6 — Polish for launch — not started

---

## License

MIT. See `LICENSE`.
