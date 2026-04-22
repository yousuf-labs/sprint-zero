# Sprint Zero

**Point it at a product. Answer three questions. Get back a complete spec set and a working app.**

Sprint Zero is a Claude Code kit that gives you a full sub-agent product team. You bring the idea and a reference URL. Sprint Zero handles scoping, research, specs, parallel engineering, and QA — then hands you a running product.

Built for PMs, founders, and anyone who wants to validate an idea end-to-end without spinning up an engineering team.

---

## What it does

```mermaid
flowchart LR
    A[You give a URL\nand answer 3 questions] --> B[Sprint Zero generates\na complete spec set]
    B --> C[Backend and frontend\nbuild in parallel]
    C --> D[QA validates the\nfull auth dance and\ncore product loop]
    D --> E[Running product\nready to demo]
```

One command. The whole flow runs automatically.

---

## How it works — the agent team

Sprint Zero uses four sub-agents. You only talk to the orchestrator. The rest happens behind the scenes.

```mermaid
flowchart TD
    User["You\n/sprint-zero &lt;url&gt;"] --> Orchestrator["Main session\norchestrator"]

    Orchestrator --> Spec["Spec pipeline\n6 slash commands run in sequence"]
    Spec --> TL["tech-lead\nReads specs, produces build brief"]
    TL --> Orchestrator

    Orchestrator --> BE["backend-engineer\nExpress + Supabase API"]
    Orchestrator --> FE["frontend-engineer\nReact + Vite + Supabase Auth"]

    BE --> QA["qa-engineer\nPlaywright auth dance\n+ core loop tests"]
    FE --> QA

    QA --> Orchestrator
    Orchestrator --> Delivery["Delivery summary\n+ running product"]
```

- **tech-lead** reads your spec set and returns a structured build brief. It does not write code.
- **backend-engineer** and **frontend-engineer** build in parallel from the same spec set. They never talk to each other — the API contract is the shared interface.
- **qa-engineer** runs the full auth dance and the core product loop end-to-end using a real browser via Playwright MCP.

---

## The spec pipeline

Before any code is written, Sprint Zero generates six documents. Each one feeds the next.

```mermaid
flowchart LR
    S1["docs/scope.md\nLevel + core loop\n+ excludes"]
    S2["docs/reference-brief.md\nWhat the reference product\ndoes and how"]
    S3["docs/prd.md\nWhat we're building\nand why"]
    S4["docs/decisions.md\nEvery scope cut\nexplained"]
    S5["docs/user-stories.md\nAcceptance criteria\nPlaywright can drive"]
    S6["docs/api-contract.md\nThe shared interface\nboth engineers build to"]

    S1 --> S2 --> S3 --> S4 --> S5 --> S6
```

These files live in `docs/`. You can read, edit, or regenerate any of them individually. The spec pipeline is resumable — if a step fails, re-running `/sprint-zero` picks up from where it stopped.

---

## The three scope levels

Pick one when you answer the scoping question. It controls everything downstream.

| Level | What it produces | Good for |
|---|---|---|
| `clickable` | Mock backend, fake data, no auth | Pitching, flow reviews, stakeholder demos |
| `working demo` | Real Supabase, real auth, one core loop works end-to-end | Proving the idea actually works |
| `pilot-ready` | Working demo + error states, input validation, loading states | Handing to 5–10 real users |

The scope level is stored in `docs/scope.md` and drives every agent's behaviour. The backend engineer builds differently. The QA engineer tests differently. The decisions document explains every cut in terms of the level you chose.

---

## The fixed stack

Sprint Zero v1 produces one stack. Not configurable, not inferred.

| Layer | Technology |
|---|---|
| Frontend | React + Vite |
| Backend | Express (Node.js) |
| Database + Auth | Supabase (Postgres + Supabase Auth) |
| Testing | Playwright via Playwright MCP |

You bring your own Supabase project. The free tier works fine. Setup takes about five minutes.

---

## Before you start — Supabase setup

Sprint Zero needs a live Supabase project before any build can run. This is a one-time setup per project.

### Step 1 — Create a Supabase project

Go to [supabase.com](https://supabase.com), click **New project**, and choose any name and region. Wait for it to provision (about 30 seconds).

### Step 2 — Enable email auth

In your project dashboard, go to **Authentication → Providers → Email** and make sure it is toggled on. It usually is by default.

### Step 3 — Collect your credentials

You need four values. They all live in your project's **Settings** page.

| Where to find it | What to copy | Key name in `.env` |
|---|---|---|
| Settings → API → Project URL | `https://xxxx.supabase.co` | `SUPABASE_URL` |
| Settings → API Keys → anon / public | `sb_publishable_...` | `SUPABASE_PUBLISHABLE_KEY` |
| Settings → API Keys → service_role (click Reveal) | `sb_secret_...` | `SUPABASE_SECRET_KEY` |
| Settings → Database → Connection string → URI (Session mode, port 5432) | `postgresql://postgres...` | `DATABASE_URL` |

> The `DATABASE_URL` is the direct Postgres connection string. Sprint Zero uses it to create the database tables automatically — no manual SQL pasting required.

### Step 4 — Create your `.env` file

From the repo root:

```bash
cp .env.example .env
```

Open `.env` and fill in the four values you collected above.

---

## Quick start

```bash
# 1. Clone the repo
git clone https://github.com/yousuf-alvi/sprint-zero.git
cd sprint-zero

# 2. Set up your .env (see Supabase setup above)
cp .env.example .env
# fill in the four values

# 3. Open in Claude Code
claude .

# 4. Run Sprint Zero with your reference URL
/sprint-zero https://example.com
```

Sprint Zero will ask you three scoping questions in one message. Answer in a paragraph — no need to format.

```
Before we kick off, three things:

1. What level are we building?
   clickable / working demo / pilot-ready

2. What's the core loop?
   The one user flow that must work.

3. Anything to exclude?
   Features from the reference you don't want.
```

Then it runs the full pipeline automatically. For a `working demo`, expect 10–20 minutes end-to-end.

### Starting the servers when the build is done

```bash
# Terminal 1 — backend
cd server
node seed.js       # creates tables + seeds demo data (first run only)
node index.js      # starts the API on http://localhost:3001

# Terminal 2 — frontend
cd client
npm run dev        # starts the app on http://localhost:5173
```

Open [http://localhost:5173](http://localhost:5173) in your browser.

---

## File structure

```
sprint-zero/
├── .claude/
│   ├── commands/          ← slash commands (the spec pipeline)
│   │   ├── sprint-zero.md         ← main orchestrator command
│   │   ├── sprint-zero-scope.md   ← scoping question + docs/scope.md
│   │   ├── explain-me-a-repo.md   ← research + docs/reference-brief.md
│   │   ├── prd-generator.md       ← docs/prd.md
│   │   ├── decisions-writer.md    ← docs/decisions.md
│   │   ├── user-story-writer.md   ← docs/user-stories.md
│   │   └── api-contract-writer.md ← docs/api-contract.md
│   └── agents/            ← sub-agents (the build layer)
│       ├── tech-lead.md
│       ├── backend-engineer.md
│       ├── frontend-engineer.md
│       └── qa-engineer.md
├── docs/                  ← generated spec files (created at runtime)
├── server/                ← Express + Supabase backend (created at runtime)
│   ├── migrations/
│   │   └── 001_init.sql   ← database schema
│   ├── migrate.js         ← runs the schema against your Supabase project
│   ├── seed.js            ← creates tables + demo data (runs migrate first)
│   ├── middleware/
│   │   └── auth.js        ← JWT verification via Supabase JWKS
│   ├── routes/            ← one file per API resource
│   ├── index.js           ← Express server entry point (port 3001)
│   └── .env.example
├── client/                ← React + Vite frontend (created at runtime)
├── .env.example           ← copy this to .env and fill in credentials
├── CLAUDE.md              ← project instructions for Claude Code
└── plan.md                ← phased build plan for this repo itself
```

---

## Running `node seed.js` — what it does

`seed.js` does three things in sequence:

```mermaid
flowchart LR
    A["node seed.js"] --> B["migrate.js runs\n001_init.sql via pg\nCreates tables if\nthey don't exist"]
    B --> C["Checks if demo user\nexists in Supabase Auth\nCreates if missing"]
    C --> D["Clears and re-inserts\nseed data for the\ndemo user"]
    D --> E["Prints demo\ncredentials to stdout"]
```

It is **idempotent** — running it twice is safe. Tables are created with `IF NOT EXISTS`. The demo user is checked before creation. Seed rows are cleared and re-inserted on every run.

---

## The API contract — why it matters

The API contract (`docs/api-contract.md`) is the shared interface between the backend and frontend. Both engineers build against it in parallel without talking to each other. If either deviates, the build breaks.

```mermaid
flowchart TD
    Contract["docs/api-contract.md\nEndpoint paths, request shapes,\nresponse shapes, status codes"]

    Contract --> BE["backend-engineer\nImplements every endpoint\nexactly as specified"]
    Contract --> FE["frontend-engineer\nCalls every endpoint\nexactly as specified"]
    Contract --> QA["qa-engineer\nValidates every endpoint\nagainst the contract"]
```

The contract is law. Agents are instructed never to deviate from it without updating the file first.

---

## Resuming after a failure

`/sprint-zero` is resumable. Each spec step checks whether its output file already exists before running. If a step fails, re-run `/sprint-zero <url>` and it picks up from where it stopped.

| Named failure state | What it means | How to recover |
|---|---|---|
| `STATE: SCOPE_NEEDED` | `docs/scope.md` was not written | Re-run `/sprint-zero <url>` with a reachable URL |
| `STATE: DISCOVERY_NEEDED` | Reference brief failed | Fix connectivity, re-run `/sprint-zero <url>` |
| `STATE: SPEC_INCOMPLETE` | A spec file is missing | Run the failing command directly (`/prd-generator` etc.), then re-run |
| `STATE: BUILD_BRIEF_NEEDED` | tech-lead found a doc problem | Fix the flagged issue, re-invoke tech-lead manually |
| `STATE: BUILD_NEEDED` | An engineer sub-agent failed | Re-spawn the failing engineer, then spawn qa-engineer |
| `STATE: QA_NEEDED` | Tests failed | Fix the issues reported by QA, re-spawn qa-engineer |

### Useful flags

| Flag | What it does |
|---|---|
| `--fresh` | Deletes `docs/` and regenerates all specs from scratch |
| `--rebuild` | Deletes `server/` and `client/` and rebuilds from existing specs |

Example: `/sprint-zero https://example.com --fresh` starts the whole flow over.

---

## Running a spec step individually

Every spec command can be run on its own if you want to regenerate one piece without rerunning everything.

```
/sprint-zero-scope    ← re-run the scoping question
/explain-me-a-repo    ← re-research the reference URL
/prd-generator        ← regenerate the PRD
/decisions-writer     ← regenerate the decisions log
/user-story-writer    ← regenerate user stories
/api-contract-writer  ← regenerate the API contract
```

Each command reads its inputs from the `docs/` files that already exist. Delete the specific output file first if you want the command to regenerate it.

---

## Troubleshooting

### The contacts page shows "Failed to load contacts"

The database tables don't exist yet. Run:

```bash
cd server
node seed.js
```

`seed.js` creates the tables automatically before inserting data. If it fails with a connection error, check that `DATABASE_URL` is set correctly in `server/.env`.

### The server crashes immediately on startup

Check that `server/.env` exists and has all four required values. The server will print a clear error message naming the missing key.

### JWT verification is failing (all API calls return 401)

This happened on the first build because of a Supabase-specific gotcha: new Supabase projects issue **ES256** tokens, not RS256. The auth middleware accepts both. If you're seeing 401s on a fresh Supabase project, check that `middleware/auth.js` has `algorithms: ['RS256', 'ES256']` and the JWKS URI is `{SUPABASE_URL}/auth/v1/.well-known/jwks.json` (not `/auth/v1/jwks`).

### QA browser tests didn't run

QA uses Playwright via the Playwright MCP server. If it isn't registered in your Claude Code settings, the browser tests will be skipped. Register it under the name `playwright` in `.claude/settings.json`, then re-spawn the qa-engineer agent.

### `node seed.js` says "relation does not exist"

The `DATABASE_URL` in `server/.env` is missing or wrong. Get it from Supabase Dashboard → Settings → Database → Connection string → URI. Choose **Session mode** (port 5432), not Transaction mode. It should look like:

```
postgresql://postgres.[ref]:[password]@aws-0-[region].pooler.supabase.com:5432/postgres
```

---

## What's not in v1

These are deliberate cuts, tracked in `plan.md` for v2:

- Multiple stacks (Next.js, Python/Django, Vue)
- Deployment automation — Sprint Zero produces code, you deploy it
- Multi-user workspaces in the generated product
- Payment / Stripe integration
- File upload beyond Supabase Storage defaults
- Hosted / cloud version of Sprint Zero itself

---

## Status

- Phase 1 — Foundation — complete
- Phase 2 — Scoping and discovery layer — complete
- Phase 3 — Build layer — complete
- Phase 4 — Kickoff orchestrator — complete
- Phase 5 — Mini Twenty worked example — not started
- Phase 6 — Polish for launch — not started

---

## License

MIT. See `LICENSE`. Built by [Yousuf Alvi](https://github.com/yousuf-alvi).
