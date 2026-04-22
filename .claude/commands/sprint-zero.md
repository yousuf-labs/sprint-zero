# /sprint-zero

You are the Sprint Zero orchestrator. Your job: take a company URL (and optional repo URL) and run the full flow — scoping, discovery, spec generation, build briefing, parallel engineering, and QA — producing a working product from a single command.

## Arguments

`$ARGUMENTS` may contain URLs, flags, or nothing:

- First URL (optional if running interactively): the company or product being referenced
- Second URL (optional): a specific repo to reverse-engineer
- `--fresh` (optional flag): delete `docs/` before starting so all spec steps run fresh
- `--rebuild` (optional flag): delete `server/` and `client/` before the build phase, forcing a clean rebuild
- `--no-launch` (optional flag): skip the auto-launch step at the end. The user will start the servers manually.

If `$ARGUMENTS` is empty, trigger the guided intake (see Step 1). If URLs are present, run non-interactively as described.

## Resumability

Before each spec-writing step, check whether its expected output file already exists in `docs/`. If it does, print a skip message and move to the next step. This makes `/sprint-zero` safe to re-run after a partial failure — it picks up from where the previous run stopped.

If `--fresh` was passed, delete `docs/` before step 2 so all spec steps run from scratch.

If `--rebuild` was passed, delete `server/` and `client/` before step 9 so the full build runs from scratch.

## Named failure states

If any step fails unrecoverably, print the named state and a recovery instruction, then stop. Do not continue past a failure — a broken spec will produce a broken build.

```
STATE: SCOPE_NEEDED       — re-run /sprint-zero <url> with a reachable URL, or run /sprint-zero-scope <url> directly then re-run /sprint-zero <url>
STATE: DISCOVERY_NEEDED   — fix connectivity or Brave MCP access, then re-run /sprint-zero <url> to resume
STATE: SPEC_INCOMPLETE    — run the failing command directly (/prd-generator, /decisions-writer, etc.), then re-run /sprint-zero <url> to resume from the build step
STATE: BUILD_BRIEF_NEEDED — fix the flagged doc issue reported by tech-lead, then re-invoke the tech-lead sub-agent manually
STATE: BUILD_NEEDED       — re-spawn the failing engineer sub-agent directly, then spawn qa-engineer once both engineers complete
STATE: QA_NEEDED          — fix the contract mismatches or test failures reported by qa-engineer, then re-spawn qa-engineer
```

---

## Step 1 — Validate and prepare

**If `$ARGUMENTS` contains URLs**, extract the first URL as the company URL and the second (if present) as the repo URL. If either URL does not start with `http://` or `https://`, prepend `https://` before proceeding. Skip to the `--fresh` / `--rebuild` handling below.

**If `$ARGUMENTS` is empty**, ask the user exactly this, in one message:

> Before we kick off, three things — answer in a paragraph, no need to format:
>
> 1. **Project name** — a short slug for this build (e.g. `linear-clone`, `mini-crm`). Used to label the delivery summary.
> 2. **Company or product URL** — the reference you want Sprint Zero to study.
> 3. **Repo URL** — a specific repo to reverse-engineer alongside the product site, or "none".

Wait for the reply. Parse it for the three fields. If a URL is given without a scheme, prepend `https://`. Store the project name for use in the delivery summary. Use the company URL and optional repo URL as inputs to all downstream steps, exactly as if they had been passed as `$ARGUMENTS`.

If `--fresh` was passed, delete the `docs/` directory and confirm deletion before continuing.

If `--rebuild` was passed, delete `server/` and `client/` and confirm deletion before continuing.

Print:

> Sprint Zero starting. Project: [project name or URL if no name given]. Scope level will be set in step 2.

---

## Step 1b — Supabase preflight

Before the scoping conversation begins, check whether a `.env` file exists at the project root and contains non-empty values for `SUPABASE_URL`, `SUPABASE_PUBLISHABLE_KEY`, `SUPABASE_SECRET_KEY`, and `DATABASE_URL`.

If the file is missing or any key is empty, print this block exactly and wait for the user to confirm before continuing:

> **Supabase setup required before we build.**
>
> Sprint Zero needs a live Supabase project. Here's what to do — takes about 3 minutes:
>
> 1. Go to [supabase.com](https://supabase.com) → New project. Choose any name and region.
> 2. Once created, go to **Settings → API**. Copy:
>    - **Project URL** → `SUPABASE_URL`
>    - **anon / public** key → `SUPABASE_PUBLISHABLE_KEY`
>    - **service_role** key → `SUPABASE_SECRET_KEY`
> 3. Go to **Settings → Database → Connection string → URI** (choose Session mode, port 5432). Copy the full URL → `DATABASE_URL`. It looks like: `postgresql://postgres.[ref]:[password]@aws-0-[region].pooler.supabase.com:5432/postgres`
> 4. Go to **Authentication → Providers → Email** and make sure it is enabled.
> 5. Create a `.env` file at the project root (copy `.env.example` and fill in all four values).
>
> Reply "ready" when done and Sprint Zero will continue.

After the user replies "ready", re-check `.env`. If it still looks empty, print the same block again and wait. Do not proceed to scoping until all four keys are present.

If the file exists and all four keys are non-empty, print:

> `.env` found with Supabase credentials. Continuing.

Note: database tables are created automatically when `node seed.js` runs — no manual SQL pasting required.

---

## Step 2 — Scoping

Check whether `docs/scope.md` exists. If it does, print:

> `docs/scope.md` already exists — skipping step 2. Delete it or use --fresh to regenerate.

If it does not exist:

Read `.claude/commands/sprint-zero-scope.md` and follow its instructions exactly. Do not summarise or simulate — read the actual file and do what it says. Pass the URLs from `$ARGUMENTS` as the arguments that command expects.

If `docs/scope.md` is not written by the end of this step, print `STATE: SCOPE_NEEDED` with recovery instructions and stop.

---

## Step 3 — Reference brief

Check whether `docs/reference-brief.md` exists. If it does, print:

> `docs/reference-brief.md` already exists — skipping step 3. Delete it or use --fresh to regenerate.

If it does not exist:

Read `.claude/commands/explain-me-a-repo.md` and follow its instructions exactly. Do not summarise or simulate — read the actual file and do what it says. Pass the URLs from `$ARGUMENTS` as the arguments that command expects.

If `docs/reference-brief.md` is not written by the end of this step, print `STATE: DISCOVERY_NEEDED` with recovery instructions and stop.

---

## Step 4 — PRD

Check whether `docs/prd.md` exists. If it does, print:

> `docs/prd.md` already exists — skipping step 4. Delete it or use --fresh to regenerate.

If it does not exist:

Read `.claude/commands/prd-generator.md` and follow its instructions exactly. Do not summarise or simulate — read the actual file and do what it says.

If `docs/prd.md` is not written by the end of this step, print `STATE: SPEC_INCOMPLETE` with recovery instructions and stop.

---

## Step 5 — Decisions

Check whether `docs/decisions.md` exists. If it does, print:

> `docs/decisions.md` already exists — skipping step 5. Delete it or use --fresh to regenerate.

If it does not exist:

Read `.claude/commands/decisions-writer.md` and follow its instructions exactly. Do not summarise or simulate — read the actual file and do what it says.

If `docs/decisions.md` is not written by the end of this step, print `STATE: SPEC_INCOMPLETE` with recovery instructions and stop.

---

## Step 6 — User stories

Check whether `docs/user-stories.md` exists. If it does, print:

> `docs/user-stories.md` already exists — skipping step 6. Delete it or use --fresh to regenerate.

If it does not exist:

Read `.claude/commands/user-story-writer.md` and follow its instructions exactly. Do not summarise or simulate — read the actual file and do what it says.

If `docs/user-stories.md` is not written by the end of this step, print `STATE: SPEC_INCOMPLETE` with recovery instructions and stop.

---

## Step 7 — API contract

Check whether `docs/api-contract.md` exists. If it does, print:

> `docs/api-contract.md` already exists — skipping step 7. Delete it or use --fresh to regenerate.

If it does not exist:

Read `.claude/commands/api-contract-writer.md` and follow its instructions exactly. Do not summarise or simulate — read the actual file and do what it says.

If `docs/api-contract.md` is not written by the end of this step, print `STATE: SPEC_INCOMPLETE` with recovery instructions and stop.

---

## Step 8 — Build brief

Print:

> Spec set complete. Invoking tech-lead for the build brief.

Invoke the `tech-lead` sub-agent. The one load-bearing instruction in the prompt is that the contract is law and deviations are not allowed. Tech-lead reads `docs/` itself to get the scope level, core loop, and the full spec set.

If tech-lead reports a missing or malformed doc, print `STATE: BUILD_BRIEF_NEEDED` with recovery instructions and stop.

Once tech-lead returns its build brief, print it so the user can read it before the build starts.

---

## Step 9 — Parallel build and QA

Print:

> Build brief received. Spawning backend-engineer and frontend-engineer in parallel.

Spawn `backend-engineer` and `frontend-engineer` in the same turn (two Task calls in parallel). Pass the scope level and core loop in each prompt. The one load-bearing instruction: the contract is law and deviations are not allowed. The workers read `docs/` themselves via relative paths.

Wait for both to return their exact completion messages:
- Backend: "Backend complete. All endpoints match docs/api-contract.md."
- Frontend: "Frontend complete. All API calls match docs/api-contract.md."

If either returns anything other than its completion message, print `STATE: BUILD_NEEDED` with recovery instructions and stop.

Once both complete, print:

> Backend and frontend complete. Spawning qa-engineer.

Spawn `qa-engineer`. Pass the scope level and core loop in the prompt. Confirm that backend runs on port 3001 and frontend on port 5173. For `clickable` scope, instruct QA to skip the auth dance and API integration tests.

---

## Step 10 — Delivery

Once QA returns its report, print the delivery summary using this template:

```
SPRINT ZERO — DELIVERY SUMMARY
==============================
Project: <name from step 1, or URL if no name was collected>
Scope level: <level>
Core loop: <from docs/scope.md>
Backend: PASS / FAIL
Frontend: PASS / FAIL
Auth dance (signup → session → logout → login → protected → 401): PASS / FAIL / N/A
QA integration tests: X/X passed
QA browser tests: X/X passed
Known issues: <list any, or "none">
Ready to demo: YES / NO
```

If QA reports failures, print `STATE: QA_NEEDED` with recovery instructions after the summary.

If the QA report mentions that database tables are missing or the seed script failed, print this reminder:

> **Database setup needed — likely a missing `DATABASE_URL`:**
> 1. Go to Supabase Dashboard → Settings → Database → Connection string → URI (Session mode, port 5432).
> 2. Copy the full URL and add it to `server/.env` as `DATABASE_URL=...`
> 3. Run `cd server && node seed.js` — this creates the tables automatically and populates demo data.
> 4. Restart the server and re-run QA.

---

## Step 11 — Auto-launch the app

If `--no-launch` was passed, or if QA reported `Ready to demo: NO`, skip this step and print:

> Skipping auto-launch. Start the servers manually — see the "Start the app manually" section in the README.

Otherwise, hand the user a working app without making them wire it up themselves.

**11a. Install dependencies if needed.** Check whether `server/node_modules` and `client/node_modules` exist. If either is missing, run `npm install` in that folder. Print one line per install ("Installing server dependencies…" / "Installing client dependencies…"). QA may have already done this — don't re-run if the folder exists.

**11b. Wire `client/.env`.** If `client/.env` does not exist, create it from `client/.env.example`, copying `SUPABASE_URL` from the root `.env` into `VITE_SUPABASE_URL` and `SUPABASE_PUBLISHABLE_KEY` into `VITE_SUPABASE_PUBLISHABLE_KEY`. If `client/.env.example` is missing, write `client/.env` directly with those two lines.

**11c. Seed the database.** Run `cd server && node seed.js`. This is idempotent — safe to re-run after QA. Capture the demo user's email and password from stdout for the launch summary.

**11d. Start the backend in the background.** Run `cd server && node index.js` as a background process. Wait two seconds, then verify with `curl -s -o /dev/null -w "%{http_code}" http://localhost:3001/` (any 2xx/3xx/4xx response means it's up; a connection error means it failed to start).

**11e. Start the frontend in the background.** Run `cd client && npm run dev` as a background process. Wait three seconds, then verify with `curl -s -o /dev/null -w "%{http_code}" http://localhost:5173/`.

**11f. Print the launch summary** in this exact format:

```
SPRINT ZERO — APP IS RUNNING
============================
Backend:  http://localhost:3001  (background)
Frontend: http://localhost:5173  (background)

Demo login:
  email:    <from seed output>
  password: <from seed output>

Open http://localhost:5173 in your browser.

To stop the servers later:
  pkill -f "node index.js" && pkill -f "vite"
```

If any sub-step in 11 fails, print `STATE: LAUNCH_FAILED — start the servers manually per the README.` and stop. Do not retry. The build succeeded — only the launch did not.

<!-- $ARGUMENTS is the interpolation token Claude Code replaces with everything the user typed after /sprint-zero. It must appear in the file for URL and flag values to be accessible throughout these instructions. -->
$ARGUMENTS
