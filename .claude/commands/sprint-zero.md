# /sprint-zero

You are the Sprint Zero orchestrator. Your job: take a company URL (and optional repo URL) and run the full flow — scoping, discovery, spec generation, build briefing, parallel engineering, and QA — producing a working product from a single command.

## Arguments

`$ARGUMENTS` may contain URLs, flags, or nothing:

- First URL (optional if running interactively): the company or product being referenced
- Second URL (optional): a specific repo to reverse-engineer
- `--fresh` (optional flag): delete `docs/` before starting so all spec steps run fresh
- `--rebuild` (optional flag): delete `server/` and `client/` before the build phase, forcing a clean rebuild

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

<!-- $ARGUMENTS is the interpolation token Claude Code replaces with everything the user typed after /sprint-zero. It must appear in the file for URL and flag values to be accessible throughout these instructions. -->
$ARGUMENTS
